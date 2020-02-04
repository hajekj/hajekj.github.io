---
id: 187
title: Building custom Docker images for use in App Service on Linux
date: 2016-12-25T21:00:29+01:00
author: Jan Hajek
layout: post
guid: https://hajekj.net/?p=187
permalink: /2016/12/25/building-custom-docker-images-for-use-in-app-service-on-linux/
categories:
  - English
  - Microsoft
  - Microsoft Azure
  - Open Source
tags:
  - App Service on Linux
  - Containers
  - Docker
  - HHVM
---
<!-- wp:paragraph {"coblocks":[]} -->
<p>If you have been <a href="https://connectevent.microsoft.com/">watching Connect();</a> or if you read the <a href="https://azure.microsoft.com/en-us/blog/">Azure blog</a>, you should be aware by now, that Azure has recently introduced a pretty cool service called <a href="https://docs.microsoft.com/en-us/azure/app-service/app-service-linux-readme">App Service on Linux</a>. It is probably also not going to be a surprise to you if I tell you that it is being powered by Docker. And that is not the only great thing about it - you can also bring your own Docker image into App Service on Linux!</p>
<!-- /wp:paragraph -->

<!-- wp:more {"coblocks":[]} -->
<!--more-->
<!-- /wp:more -->

<!-- wp:paragraph {"coblocks":[]} -->
<p><em>I might use&nbsp;<strong>ASL</strong> shortcut sometimes which refers to&nbsp;<strong>A</strong>pp&nbsp;<strong>S</strong>ervice on&nbsp;<strong>L</strong>inux and&nbsp;<strong>ASW</strong> which refers to&nbsp;<strong>A</strong>pp&nbsp;<strong>S</strong>ervice on&nbsp;<strong>W</strong>indows.</em></p>
<!-- /wp:paragraph -->

<!-- wp:paragraph {"coblocks":[]} -->
<p><em>This article also assumes you have some elementary knowledge of Containers and Docker. If you need some quickstart, I suggest you head over to <a href="https://mva.microsoft.com/en-US/training-courses/exploring-microservices-in-docker-and-microsoft-azure-11796?l=FhWy8pmEB_004984382">this course at Microsoft Virtual Academy</a>.</em></p>
<!-- /wp:paragraph -->

<!-- wp:heading {"level":1,"coblocks":[]} -->
<h1>Why is this so super cool?</h1>
<!-- /wp:heading -->

<!-- wp:paragraph {"coblocks":[]} -->
<p>One of the questions you may have is "Why do you say it is important when there is Azure Container Service?" The answer to that is very simple and very straightforward: Take the power of App Service on Windows with all of its services (scalability, custom domains, continuous integration, ...) and leverage its power&nbsp;with your custom environment thanks to Docker.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph {"coblocks":[]} -->
<p>This also allows you to rapidly deploy and scale custom containers and solutions built-in Docker.</p>
<!-- /wp:paragraph -->

<!-- wp:heading {"level":1,"coblocks":[]} -->
<h1>Building your own</h1>
<!-- /wp:heading -->

<!-- wp:paragraph {"coblocks":[]} -->
<p>As such as it might seem, you can make use of the default containers (at the time of writing we have PHP, Node and .NET Core, all in multiple versions). However, you can also make use of your own in order to use a specific version, module or even custom runtime. In this article, we are going to take <a href="http://hhvm.com/">Facebook's HHVM</a>&nbsp;and run it on App Service on Linux.</p>
<!-- /wp:paragraph -->

<!-- wp:heading {"coblocks":[]} -->
<h2>Reusing existing&nbsp;containers</h2>
<!-- /wp:heading -->

<!-- wp:paragraph {"coblocks":[]} -->
<p>You may ask - why do we need to build a custom container when there is already a <a href="https://hub.docker.com/r/hhvm/hhvm/">HHVM image on Docker hub</a>, can't we just use it? The answer is no. The Docker image is just basically the runtime, nothing else.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph {"coblocks":[]} -->
<p>Theoretically, you could take an existing container image and use it directly on App Service on Linux, however, you may not be able to have logging configured correctly or you may not have access to the filesystem where your application is.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph {"coblocks":[]} -->
<p>If you already have experience with Docker, you know you can either package your application directly into the image or you can mount it in (so Docker basically serves as environment container).</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph {"coblocks":[]} -->
<p>On ASL you can make use of both types. You can for example take the <a href="https://hub.docker.com/r/clue/adminer/">Adminer&nbsp;image</a> and deploy it and everything will work out of box. But then, you won't be able to access the files in the container, not even using FTP.</p>
<!-- /wp:paragraph -->

<!-- wp:heading {"coblocks":[]} -->
<h2>Storage in containers</h2>
<!-- /wp:heading -->

<!-- wp:paragraph {"coblocks":[]} -->
<p>So like I mentioned above, you can either pack your application directly into the container, which results in sort of <a href="https://docs.microsoft.com/en-us/azure/app-service/app-service-local-cache">App Service Local Cache</a> equivalent when using ASW. In this case, you will be probably using some sort of continuous integration system like VSTS to automatically build your containers (remember that without a storage connection, any filesystem write will not be persistent).</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph {"coblocks":[]} -->
<p>However, in this case, we would like to leverage the regular App Service Filesystem, so we can interact with the application using FTP. When a container is deployed, ASL mounts the equivalent of&nbsp;<em>D:\home</em> path on ASW to&nbsp;<em>/home</em> (using <a href="https://docs.docker.com/engine/tutorials/dockervolumes/">volume mount in Docker</a>). Now when that happens, it is up to your container to map the corresponding paths into the application. In order to understand how this works more closely, take a look at the <a href="https://github.com/appsvc/php/blob/master/7.0.6-apache/Dockerfile">official Dockerfile</a> used in PHP7 container on ASL.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph {"coblocks":[]} -->
<p>The important part there is following:</p>
<!-- /wp:paragraph -->

<!-- wp:coblocks/gist {"url":"https://gist.github.com/hajekj/17ab3a7a18b1ad545ff000252dc35451","file":"187.1.Dockerfile","coblocks":[]} -->
<div class="wp-block-coblocks-gist"><script src="https://gist.github.com/hajekj/17ab3a7a18b1ad545ff000252dc35451.js?file=187.1.Dockerfile"></script><noscript><a href="https://gist.github.com/hajekj/17ab3a7a18b1ad545ff000252dc35451#file-187-1.Dockerfile">View this gist on GitHub</a></noscript></div>
<!-- /wp:coblocks/gist -->

<!-- wp:paragraph {"coblocks":[]} -->
<p>What these two lines actually do is that they create a symbolic link between the&nbsp;<em>/home/site/wwwroot</em> and the default Apache's <em>/var/www/html</em>&nbsp;folder. You could also take a different approach and for example specify these paths in the&nbsp;<em>apache2.conf</em> for the virtual host.</p>
<!-- /wp:paragraph -->

<!-- wp:heading {"coblocks":[]} -->
<h2>Making your own Dockerfile</h2>
<!-- /wp:heading -->

<!-- wp:paragraph {"coblocks":[]} -->
<p>Since we now know how the storage works in ASL, we are going to create our own Dockerfile. We will make use of <a href="https://github.com/hhvm/hhvm-docker/tree/master/hhvm-latest-proxygen">one of HHVM's default images</a> which is using <a href="https://docs.hhvm.com/hhvm/basic-usage/proxygen">Proxygen</a>&nbsp;as its web server.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph {"coblocks":[]} -->
<p><em>On a side note, Proxygen will never be exposed directly to Internet, there is of course a loadbalancer in front of it just like with ASW, which for example handles SSL termination</em><em>.</em></p>
<!-- /wp:paragraph -->

<!-- wp:paragraph {"coblocks":[]} -->
<p>Like mentioned above in the storage part, all we have to do to make this work is to map the file system paths correctly, but unlike in the official images which using symbolic links, we are going to point directly to the mapped volume. Which means that we are not going to be changing the&nbsp;<em>Dockerfile</em> itself, but we will be changing&nbsp;<em>server.ini</em> file which serves as the configuration. After you add the correct paths to the configuration and also specify the log file, the&nbsp;<em>server.ini</em> should look like so:</p>
<!-- /wp:paragraph -->

<!-- wp:coblocks/gist {"url":"https://gist.github.com/hajekj/17ab3a7a18b1ad545ff000252dc35451","file":"187-2.ini","coblocks":[]} -->
<div class="wp-block-coblocks-gist"><script src="https://gist.github.com/hajekj/17ab3a7a18b1ad545ff000252dc35451.js?file=187-2.ini"></script><noscript><a href="https://gist.github.com/hajekj/17ab3a7a18b1ad545ff000252dc35451#file-187-2-ini">View this gist on GitHub</a></noscript></div>
<!-- /wp:coblocks/gist -->

<!-- wp:paragraph {"coblocks":[]} -->
<p>You can also find full <a href="https://docs.hhvm.com/hhvm/configuration/INI-settings">INI reference here</a>&nbsp;in case you need to make more changes.</p>
<!-- /wp:paragraph -->

<!-- wp:heading {"coblocks":[]} -->
<h2>Building and publishing the container</h2>
<!-- /wp:heading -->

<!-- wp:paragraph {"coblocks":[]} -->
<p>Now all you need is to build the container and push it to to your repository, we will be using <a href="https://hub.docker.com/">Docker hub</a> for this. This is also very straightforward, so all you have to do is to first login into Docker hub:</p>
<!-- /wp:paragraph -->

<!-- wp:coblocks/gist {"url":"https://gist.github.com/hajekj/17ab3a7a18b1ad545ff000252dc35451","file":"187-3.sh","coblocks":[]} -->
<div class="wp-block-coblocks-gist"><script src="https://gist.github.com/hajekj/17ab3a7a18b1ad545ff000252dc35451.js?file=187-3.sh"></script><noscript><a href="https://gist.github.com/hajekj/17ab3a7a18b1ad545ff000252dc35451#file-187-3-sh">View this gist on GitHub</a></noscript></div>
<!-- /wp:coblocks/gist -->

<!-- wp:paragraph {"coblocks":[]} -->
<p>And then, when in the folder with your&nbsp;<em>Dockerfile</em> and&nbsp;<em>server.ini</em> you just need to execute Docker's build command:</p>
<!-- /wp:paragraph -->

<!-- wp:coblocks/gist {"url":"https://gist.github.com/hajekj/17ab3a7a18b1ad545ff000252dc35451","file":"187-4.sh","coblocks":[]} -->
<div class="wp-block-coblocks-gist"><script src="https://gist.github.com/hajekj/17ab3a7a18b1ad545ff000252dc35451.js?file=187-4.sh"></script><noscript><a href="https://gist.github.com/hajekj/17ab3a7a18b1ad545ff000252dc35451#file-187-4-sh">View this gist on GitHub</a></noscript></div>
<!-- /wp:coblocks/gist -->

<!-- wp:paragraph {"coblocks":[]} -->
<p>Which is going to download the dependencies and build the container image for you and then all you have to do is to publish it into the Docker hub:</p>
<!-- /wp:paragraph -->

<!-- wp:coblocks/gist {"url":"https://gist.github.com/hajekj/17ab3a7a18b1ad545ff000252dc35451","file":"187-5.sh","coblocks":[]} -->
<div class="wp-block-coblocks-gist"><script src="https://gist.github.com/hajekj/17ab3a7a18b1ad545ff000252dc35451.js?file=187-5.sh"></script><noscript><a href="https://gist.github.com/hajekj/17ab3a7a18b1ad545ff000252dc35451#file-187-5-sh">View this gist on GitHub</a></noscript></div>
<!-- /wp:coblocks/gist -->

<!-- wp:paragraph {"coblocks":[]} -->
<p>After that, you should be able to see it online on your Docker hub's profile.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph {"coblocks":[]} -->
<p>The repo with this project can be found on <strong><a href="https://github.com/TheNetworg/appsvc-hhvm">GitHub</a></strong>&nbsp;or you can deploy it directly from <strong><a href="https://hub.docker.com/r/thenetworg/appsvc-hhvm/">Docker Hub</a></strong>.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph {"coblocks":[]} -->
<p><em>In order to be able to do this, you need to have Docker installed, in order to install it you can <a href="https://docs.docker.com/engine/installation/">get started here</a>. I have been using Docker for Windows for this process. Also if you need more information about these commands, some basics can be found in the <a href="https://docs.docker.com/engine/tutorials/dockerimages/">official documentation</a>.</em></p>
<!-- /wp:paragraph -->

<!-- wp:heading {"level":1,"coblocks":[]} -->
<h1>Using custom image&nbsp;on App Service on Linux</h1>
<!-- /wp:heading -->

<!-- wp:paragraph {"coblocks":[]} -->
<p>So now with the image published we can now create a website and use our custom image in it!</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph {"coblocks":[]} -->
<p>The process is very simple - in Azure Portal you <a href="https://ms.portal.azure.com/#create/Microsoft.AppSvcLinux">create a new resource - Web App on Linux</a>&nbsp;and specify the image name you published to Docker Hub like so:</p>
<!-- /wp:paragraph -->

<!-- wp:image {"id":198,"align":"center","linkDestination":"custom","coblocks":[]} -->
<div class="wp-block-image"><figure class="aligncenter"><a href="/uploads/2016/12/appsvc-hhvm2.png"><img src="/uploads/2016/12/appsvc-hhvm2-300x295.png" alt="" class="wp-image-198"/></a></figure></div>
<!-- /wp:image -->

<!-- wp:paragraph {"coblocks":[]} -->
<p>After waiting a minute or so for the resource deployment, you can go ahead and create an&nbsp;<em>index.php</em> in the site's FTP directory (or publish to it from Git) and with just a simple use of&nbsp;<em>&lt;?php phpinfo(); ?&gt;</em> you will be able to see that we have the HHVM up and running on App Service on Linux!</p>
<!-- /wp:paragraph -->

<!-- wp:image {"id":197,"align":"center","linkDestination":"custom","coblocks":[]} -->
<div class="wp-block-image"><figure class="aligncenter"><a href="/uploads/2016/12/appsvc-hhvm.png"><img src="/uploads/2016/12/appsvc-hhvm-300x181.png" alt="" class="wp-image-197"/></a></figure></div>
<!-- /wp:image -->

<!-- wp:heading {"coblocks":[]} -->
<h2>Lifecycle</h2>
<!-- /wp:heading -->

<!-- wp:paragraph {"coblocks":[]} -->
<p>Regarding container lifecycle, especially updates - whenever you need to update the image (assuming you are using the&nbsp;<em>latest</em> version), you are going to have to restart the site for it to fetch the new version of the image. In case of you needing to change the image completely or changing the version, you would change it first in the&nbsp;<span style="text-decoration: underline;">Docker tab</span> and then restart the site as well.</p>
<!-- /wp:paragraph -->

<!-- wp:heading {"coblocks":[]} -->
<h2>Logs</h2>
<!-- /wp:heading -->

<!-- wp:paragraph {"coblocks":[]} -->
<p>As for the logs, all the Docker logs get stored into regular log path which can be accessed from FTP or Kudu (<em>/home/LogFiles/docker/</em>). From there you can find out if there were any issues starting the container or during the runtime which is very useful when debugging custom image.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph {"coblocks":[]} -->
<p>Additionally, it is always good to force the Docker image which you will be using to at least store logs into the filesystem so you can access the application logs (may it be Apache or your custom application as well) and diagnose any issues which might occur.</p>
<!-- /wp:paragraph -->

<!-- wp:heading {"level":1,"coblocks":[]} -->
<h1>Summary</h1>
<!-- /wp:heading -->

<!-- wp:paragraph {"coblocks":[]} -->
<p>Personally, I believe that App Service on Linux offers a great flexibility in being able to bring your own container as well as platform-oriented features like scaling and so on and is very useful when bringing your existing workloads to Azure. Last, but not least, it also solves the issue where Node.js native modules (written only for Linux for example)&nbsp;<a href="https://docs.microsoft.com/en-us/azure/nodejs-use-node-modules-azure-apps">didn't work or compile</a> on App Service on Windows.</p>
<!-- /wp:paragraph -->