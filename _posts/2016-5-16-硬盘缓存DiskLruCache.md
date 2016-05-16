---
layout: post
author: mrrobot97
title: 硬盘缓存DiskLruCache
---

众所周知，Google官方提供一套内存缓存方案LruCache,但是并没有提供硬盘缓存方案。第三方提供了一个硬盘缓存方案DiskLruCache，不过没有被Google官方收录，故没有加入到SDK中。要使用DiskLruCache，先下载其源码：

[DiskLruCache源码](android.googlesource.com/platform/libcore/+/jb-mr2-release/luni/src/main/java/libcore/io/DiskLruCache.java)

下载后将源码放在自己的项目中即可使用。关于DiskLruCache的使用：

1.创建实例：
DiskLruCache没有new方法，我们只能通过`DiskLruCache open(File directory, int appVersion, int valueCount, long maxSize)`方法获取其实例。参数故名思意，directory为缓存文件的存放路径，通常我们可以指定为 `/sdcard/Android/data/<application package>/cache `下。
appVersion为软件版本，通常指定1即可。注意当appVersion变更后，原来的缓存文件会全部清除，而我们一般也没有需求去变更这个版本号。valueCount通常指定为1，maxSize自己指定一个合适大小。

2.将文件缓存：
利用`DiskLruCache.Editor`将文件写入缓存，这个类同样无法new出来，需要调用`DiskLruCache.edit(String key)`方法来获取，这里的key是我们为将要缓存的文件指定的key,将来从缓存中读取文件也要使用这个key。我们使用Editor.newOutputStream(int index);方法来获取写入缓存的OutputStream，这里的index由于我们之前的valueCount传入的为1，故index应为0。在写入操作之后还要commit()才能生效，调用abort()则放弃写入操作。

3.读取缓存：
通过DiskLruCache.get(String key)方法我们将获取到一个Snapshot对象，然后我们调用Snapshot.getInputstream(int index)方法即可获取到缓存文件的InputStream了，得到了这个流我们再将其解析为我们需要的形式即可。这个的key就是上文写入缓存的key,而index同样为0。

4.移除缓存：
直接调用DiskLruCache.remove(String key)方法即可，显然这个key与上文同意。


一个使用DiskLruCache缓存Bitmap的小例子：

```

public class MainActivity extends AppCompatActivity {
    private DiskLruCache mDiskLruCache;
    private ImageView mImageView;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        init();
    }

    private void init() {
        long maxSize=Runtime.getRuntime().maxMemory()/8;
        mImageView= (ImageView) findViewById(R.id.image_view);
        File cacheFile=getDiskCacheDir(this,"bitmap");
        if (!cacheFile.exists()){
            cacheFile.mkdir();
        }
        try {
            mDiskLruCache=DiskLruCache.open(cacheFile,1,1,maxSize);
        } catch (IOException e) {
            e.printStackTrace();
        }
        cacheImage();
        loadImage();
    }

    private void loadImage() {
        String downloadUrl="http://img2.imgtn.bdimg.com/it/u=1903957143,479133575&fm=11&gp=0.jpg";
        String key=hashKeyForDisk(downloadUrl);
        DiskLruCache.Snapshot snapshot=null;
        InputStream in=null;
        try {
            snapshot=mDiskLruCache.get(key);
            if (snapshot!=null){
                in=snapshot.getInputStream(0);
                Bitmap bitmap=BitmapFactory.decodeStream(in);
                mImageView.setImageBitmap(bitmap);
                Log.d("YJW","HERE");
            }
        } catch (IOException e) {
            e.printStackTrace();
        }finally {
            if (in!=null){
                try {
                    in.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }

    }

    private void cacheImage() {
        new Thread(new Runnable() {
            @Override
            public void run() {
                String downloadUrl="http://img2.imgtn.bdimg.com/it/u=1903957143,479133575&fm=11&gp=0.jpg";
                String key=hashKeyForDisk(downloadUrl);
                DiskLruCache.Editor editor=null;
                try {
                    editor=mDiskLruCache.edit(key);
                    OutputStream out=editor.newOutputStream(0);
                    if (downloadUrlToStream(downloadUrl,out)){
                        editor.commit();
                    }else{
                        editor.abort();
                    }
                    mDiskLruCache.flush();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }).start();

    }

    //将图片url写入到outputstream中
    private boolean downloadUrlToStream(String downloadUrl, OutputStream outputStream){
        HttpURLConnection conn=null;
        BufferedOutputStream out=null;
        BufferedInputStream in=null;
        try {
            URL url=new URL(downloadUrl);
            conn= (HttpURLConnection) url.openConnection();
            in=new BufferedInputStream(conn.getInputStream());
            out=new BufferedOutputStream(outputStream);
            int b;
            while((b=in.read())!=-1){
                out.write(b);
            }
            return true;
        } catch (MalformedURLException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }finally {
            if (conn!=null){
                conn.disconnect();
            }
            try {
                if (out!=null){
                    out.close();
                }
                if (in!=null){
                    in.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        return false;
    }


    public String hashKeyForDisk(String key) {
        String cacheKey;
        try {
            final MessageDigest mDigest = MessageDigest.getInstance("MD5");
            mDigest.update(key.getBytes());
            cacheKey = bytesToHexString(mDigest.digest());
        } catch (NoSuchAlgorithmException e) {
            cacheKey = String.valueOf(key.hashCode());
        }
        return cacheKey;
    }

    private String bytesToHexString(byte[] bytes) {
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < bytes.length; i++) {
            String hex = Integer.toHexString(0xFF & bytes[i]);
            if (hex.length() == 1) {
                sb.append('0');
            }
            sb.append(hex);
        }
        return sb.toString();
    }
    public File getDiskCacheDir(Context context, String uniqueName) {
        String cachePath;
        if (Environment.MEDIA_MOUNTED.equals(Environment.getExternalStorageState())
                || !Environment.isExternalStorageRemovable()) {
            cachePath = context.getExternalCacheDir().getPath();
        } else {
            cachePath = context.getCacheDir().getPath();
        }
        return new File(cachePath + File.separator + uniqueName);
    }


}

```