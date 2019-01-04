---
layout: default
title: 微信公众号的服务器token验证
tags: [微信,微信公众号]
imgPath: 2019-01-03-WEIXIN-TOKEN
---
接入微信公众平台开发，最开始的一步就是让你验证服务器的有效性。  

官方文档：<https://mp.weixin.qq.com/wiki?t=resource/res_main&id=mp1421135319>{:target="_blank"}

![]({{site.url}}{{site.post-img-dir}}/{{page.imgPath}}/server-config.jpg)

你需要填写你的服务器项目url、token、EncodingAESKey，选择明文模式（官方推荐安全模式，需要加解密消息），其中token、EncodingAESKe可以随机填写，EncodingAESKey在这里用不到，当你点击提交的时候微信会携带signature、timestamp、nonce、echostr四个参数请求该url，而你的服务需要验证并原样返回echostr参数内容。

```
@Controller
@RequestMapping("/wx")
public class WxController {

    @RequestMapping()
    @ResponseBody
    public void checkToken(String signature, String timestamp, String nonce, String echostr
    		, HttpServletRequest request, HttpServletResponse response) throws IOException {
        
        boolean ok = MPUtils.checkToken( signature, timestamp, nonce, echostr);
        if(ok){
            response.getWriter().print(echostr);
            System.out.println("验证通过");
        }
    }

} 
```
验证步骤分为三步：
1. 将token、timestamp、nonce三个参数进行字典序排序
2. 将三个参数字符串拼接成一个字符串进行sha1加密
3. 开发者获得加密后的字符串可与signature对比，标识该请求来源于微信

```
public static boolean checkToken(String signature,String timestamp,String nonce,String echostr){

    String sortStr = sort(Constants.TOKEN, timestamp, nonce);//排序串
    String sha1Str = DigestUtils.sha1Hex(sortStr);//sha1加密串

    System.out.println("加密后："+sha1Str);

    return sha1Str.equals(signature);
}	

public static String sort(String token,String timestamp,String nonce){
    String[] arr = { token, timestamp, nonce };
    Arrays.sort(arr); // 字典序排序
    String str = arr[0] + arr[1] + arr[2];
    return str;
}
```
> 其中DigestUtils.sha1Hex(sortStr);需要引入commons-codec依赖  

```
<dependency>
	<groupId>commons-codec</groupId>
	<artifactId>commons-codec</artifactId>
	<version>$1.7</version>
</dependency>
```
部署项目运行，填写URL、token提交，例如我的url是 https://huangtl.com/wx，因为需要提供外网地址，若需要本地开发测试可参考我的另一篇文章《[内网映射工具 ngrok]({{site.url}}/2018/02/13/NGROK)》映射本地的一个地址。
> 注意：验证成功后，用户正在公众号的操作（消息、事件等）微信都会请求到该URL上，所以你在验证成功后可以将该方法注释，重新使用该路径写针对微信发送过来的消息/事件进行处理，也可以继续使用该方法做改造。

感谢阅读