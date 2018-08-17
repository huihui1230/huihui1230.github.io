---
title: Spring整合framemarker发邮件
date: 2018-08-17 10:39:02
tags: 发邮件
categories:
- 后端
- Spring
copyright: true
---
## 需求
  
系统添加一个用户之后，给这个用户发邮件，告知用户名密码等。为了不阻塞主线程，发邮件使用一个子线程来执行。  
  
<!--more-->
  
## 实现
  
Spring线程池配置：  
  
```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">

    <bean id="threadPool" class="org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor">
        <!-- 线程池维护线程的最少数量 -->
        <property name="corePoolSize" value="5"></property>
        <!--允许的空闲时间-->
        <property name="keepAliveSeconds" value="200"></property>
        <!--线程池维护线程的最大数量-->
        <property name="maxPoolSize" value="10"></property>
        <!--缓存队列-->
        <property name="queueCapacity" value="20"></property>
        <!--对拒绝task的处理策略-->
        <property name="rejectedExecutionHandler">
            <bean class="java.util.concurrent.ThreadPoolExecutor.CallerRunsPolicy"></bean>
        </property>
    </bean>
</beans>
```
  
邮件发送线程类：  
  
```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.mail.javamail.JavaMailSender;
import org.springframework.stereotype.Service;
import javax.mail.internet.MimeMessage;

@Service
public class MailThreadServiceImpl {

    @Autowired
    private JavaMailSender javaMailSender;

    public Runnable execute(MimeMessage mimeMessage) {
        return new SendMail(mimeMessage);
    }

    public class SendMail implements Runnable {

        private MimeMessage mimeMessage;

        public SendMail(MimeMessage mimeMessage) {
            this.mimeMessage = mimeMessage;
        }

        @Override
        public void run() {
            if (mimeMessage != null) {
                MailThreadServiceImpl.this.javaMailSender.send(mimeMessage);
                System.out.println("===============邮件发送成功=================");
            }
        }
    }
}
```
  
业务逻辑方法：  
  
```java
public void sendEmail(User user) {
    
    MimeMessage message = null;

    try {
        Map<String, String> data = new HashMap<>();
        data.put("logoUrl", ReadPropertiesUtil.readMailLogo());
        data.put("loginName", user.getLoginName());
        data.put("password", password);

        Template template = freeMarkerConfigurer.getConfiguration().getTemplate("mail.ftl");
        // 解析模板并替换动态数据，最终content将替换模板文件中的${content}标签。
        String htmlText = FreeMarkerTemplateUtils.processTemplateIntoString(template, data);

        message = mailSender.createMimeMessage();
        MimeMessageHelper helper = new MimeMessageHelper(message, false, "utf8");
        helper.setFrom(mailSender.getUsername());
        helper.setTo(user.getEmail());
        helper.setSubject("注册成功-邮件");
        helper.setText(htmlText, true);

        threadPoolTaskExecutor.execute(mailThreadService.execute(message));
    } catch (MessagingException e) {
        System.out.println("=========new MimeMessageHelper(message, false, \"utf8\")===========创建模板邮件失败");
    } catch (IOException e) {
        System.out.println("=========freeMarkerConfigurer.getConfiguration().getTemplate(\"mail.ftl\")==========配置模板失败");
    } catch (TemplateException e) {
        System.out.println("=========FreeMarkerTemplateUtils.processTemplateIntoString(template, data);========模板渲染动态数据异常");
    } catch (Exception e) {
        System.out.println("========================发送邮件失败==========================");
    }
}
```
  
邮件模板：  
  
mail.ftl
  
```html
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
    <title>Insert title here</title>
	<style>
		body,h1,h2,h3,h4,h5,h6,p,ul,ol,dl,dd,input,button,table,tr,td {
			margin: 0;
			padding: 0;
			outline: none;
		}

		h1,h2,h3,h4,h5,h6,b,i,em {
			font-size: 16px;
			font-weight: normal;
			font-style: normal;
		}

		a {
			color: #fff;
			text-decoration: none;
		}

		ul,ol {
			list-style: none;
			
		}

		input {
			outline: none;
		}

		input,img,select {
			vertical-align: top;
		}

		.last-bottom {
			border-bottom: none !important;
		}

		.last-right {
			/*border-right: none !important;*/
			margin-right: 0 !important;
		}

		.fl {
			float: left;
		}

		.fr {
			float: right;
		}

		/* 清除浮动 */
		.clearfix:after {
			content: "";
			display: block;
			clear: both;
		}

		.clearfix {
			zoom: 1;
		}
		
		html {
		  font-size: 8.888888886666667vw;
		}

		body {
			background: #f9f9f9;
		}

		.box {
			width: 484px;
			border-radius: 5px;
			margin: 60px auto;
		}

		.top {
			height:42px;
			background: ;
			/*background:#313D4B  url(D:/freemarker/logo.png) no-repeat center;*/
			background:#313D4B  url(${logoUrl}) no-repeat center;
			background-size:100px 42px;
		}

		.content {
			height:192px;
			background: #fff;
			overflow: hidden;
		}

		.word {
			font-size: 12px;
			color: #7c7c7c;
			margin-top: 23px;
			margin-left: 23px;
		}

		.center {
			width: 230px;
			margin: 30px auto;
			font-size: 12px;
			color: #7c7c7c;
		}

		.pass{
			margin-top: 11px;
		}

		.name {
			color: #000;
		}

		.add {
			margin-top: 11px;
		}

		.res {
			color:#4098ca;
		}

		.bottom {
			width: 260px;
			margin: 30px auto;	
			font-size: 12px;
			color: #c7c7c7;
		}

		.first {
			text-align: center;
		}

		.sec {
			width: 260px;
			text-align: center;
		}

		.reach {
			color: #4297cd;
		}

		.third {
			text-align: center;
		}
	</style>
</head>
<body>
	<div class="box">
		<div class="top"></div>
		<div class="content">
			<div class="word">欢迎使用TapWin广告投放系统,您的账户已经成功开通. </div>
			<div class="center">
				<div class="user">
					<span class="username">用户名:</span>
					<span class="name">${loginName}</span>
				</div>
				<div class="pass">
					<span class="password">密码:</span>
					<span class="name">${password}</span>
				</div>
				<div class="add">
					<span class="address">系统访问地址:</span>
					<span class="res">
						<a href="http://admin.tapwin.cn/system/user/login" style="color: blue">
						http://admin.tapwin.cn/system/user/login</a>
					</span>
				</div>
			</div>
		</div>
		<div class="bottom">
			<p class="first">如果您在使用系统时遇到任何问题可发邮件至</p>
			<p class="sec"><span class="reach">support@180.ai</span>获取帮助信息.</p>
			<p class="third">本邮件由系统自动发出,请不要回复此邮件.</p>
		</div>
	</div>
</body>
</html>
```
