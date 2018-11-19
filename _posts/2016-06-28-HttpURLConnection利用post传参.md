---
layout: post
author: mrrobot
title: HttpURLConnection利用post传参
---

直接上示例代码：

```
HttpURLConnection conn=null;
            OutputStreamWriter osw=null;
            BufferedReader br=null;
            try {
                URL url=new URL(address);
                conn= (HttpURLConnection) url.openConnection();
                conn.setDoOutput(true);
                osw=new OutputStreamWriter(conn.getOutputStream());
                osw.write("username="+userText.getText()+"&"+"password="+passText.getText());
                osw.close();
                br=new BufferedReader(new InputStreamReader(conn.getInputStream()));
                final StringBuilder response =new StringBuilder();
                String s;
                while((s=br.readLine())!=null){
                    response.append(s);
                }
                runOnUiThread(new Runnable() {
                    @Override
                    public void run() {
                        respText.setText(response);
                    }
                });
            } catch (MalformedURLException e) {
                e.printStackTrace();
            } catch (IOException e) {
                e.printStackTrace();
            }finally {
                if (br!=null){
                    try {
                        br.close();
                        if (conn!=null){
                            conn.disconnect();
                        }
                    } catch (IOException e) {
                        e.printStackTrace();
                    }
                }
            }
```
