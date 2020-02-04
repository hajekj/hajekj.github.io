---
id: 654
title: Passing state through authentication in ASP.NET Core
date: 2018-06-12T09:00:14+02:00
author: Jan Hajek
layout: post
guid: https://hajekj.net/?p=654
permalink: /2018/06/12/passing-state-through-authentication-in-asp-net-core/
categories:
  - English
  - Microsoft
  - Open Source
tags:
  - .NET
  - ASP.NET Core
  - Authentication
---
<!-- wp:paragraph {"coblocks":[]} -->
<p>When authenticating a user, you might want to persist the state through the authentication request - for example whether the user is authenticating for some special action like organizational signup or simply some state of your application. ASP.NET Core makes this very easy.</p>
<!-- /wp:paragraph -->

<!-- wp:more {"coblocks":[]} -->
<!--more-->
<!-- /wp:more -->

<!-- wp:paragraph {"coblocks":[]} -->
<p>First, you need to add the state to the request authentication request (this also works with <em>Challenge</em> in MVC Controller):</p>
<!-- /wp:paragraph -->

<!-- wp:coblocks/gist {"url":"https://gist.github.com/hajekj/17ab3a7a18b1ad545ff000252dc35451","file":"654-1.cs","coblocks":[]} -->
<div class="wp-block-coblocks-gist"><script src="https://gist.github.com/hajekj/17ab3a7a18b1ad545ff000252dc35451.js?file=654-1.cs"></script><noscript><a href="https://gist.github.com/hajekj/17ab3a7a18b1ad545ff000252dc35451#file-654-1-cs">View this gist on GitHub</a></noscript></div>
<!-- /wp:coblocks/gist -->

<!-- wp:paragraph {"coblocks":[]} -->
<p>You can see we have two dictionaries in <em>AuthenticationProperties</em>. The first one, which populates&nbsp;property called&nbsp;<em>Items</em> is actual state. The second one, which is called&nbsp;<em>Parameters</em> (I am mentioning it just to clear up the confusion) is used for adding items into the query. Thanks to parameters, you can easily add&nbsp;<em>prompt</em> property to the URL or use the&nbsp;<a href="https://hajekj.net/2017/03/06/forcing-reauthentication-with-azure-ad/"><em>max_age</em></a> parameter.</p>
<!-- /wp:paragraph -->

<!-- wp:coblocks/gist {"url":"https://gist.github.com/hajekj/17ab3a7a18b1ad545ff000252dc35451","file":"654-2.cs","coblocks":[]} -->
<div class="wp-block-coblocks-gist"><script src="https://gist.github.com/hajekj/17ab3a7a18b1ad545ff000252dc35451.js?file=654-2.cs"></script><noscript><a href="https://gist.github.com/hajekj/17ab3a7a18b1ad545ff000252dc35451#file-654-2-cs">View this gist on GitHub</a></noscript></div>
<!-- /wp:coblocks/gist -->

<!-- wp:paragraph {"coblocks":[]} -->
<p>Note, that there are many other <em>AuthenticationProperties</em> implementations like&nbsp;<a href="https://github.com/aspnet/Security/blob/8654efeb4da54e7f05dbce38ecdbcd8c540d8388/src/Microsoft.AspNetCore.Authentication.Google/GoogleChallengeProperties.cs"><em>GoogleChallengeProperties</em></a>,&nbsp;<a href="https://github.com/aspnet/Security/blob/8654efeb4da54e7f05dbce38ecdbcd8c540d8388/src/Microsoft.AspNetCore.Authentication.OpenIdConnect/OpenIdConnectChallengeProperties.cs"><em>OpenIdConnectChallengeProperties</em></a>&nbsp;etc. These are going to make working with the parameters on the IdP side really easy.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph {"coblocks":[]} -->
<p>Next, we are going to look at the state. The state is persisted in Dictionary which has a string key and string value. This makes storing larger objects more complex - you have to serialize and deserialize them as needed, however, you should keep the state as small as possible, since the identity provider or their webserver can enforce certain limits like URL length. If you need to store larger amount of data and persist it across the authentication, you can leverage <a href="https://docs.microsoft.com/en-us/aspnet/core/fundamentals/app-state?view=aspnetcore-2.1&amp;tabs=aspnetcore2x#tempdata"><em>TempData</em></a>.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph {"coblocks":[]} -->
<p>Once the state gets serialized into the request it is also going to be <a href="https://docs.microsoft.com/en-us/aspnet/core/security/data-protection/introduction?view=aspnetcore-2.1">protected</a>&nbsp;so the server doesn't manipulate it. It also contains other properties like the <em>RedirectUri</em> for example.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph {"coblocks":[]} -->
<p>Once the user authenticates and goes back to your server, you can then access the session items really easily:</p>
<!-- /wp:paragraph -->

<!-- wp:coblocks/gist {"url":"https://gist.github.com/hajekj/17ab3a7a18b1ad545ff000252dc35451","file":"654-3.cs","coblocks":[]} -->
<div class="wp-block-coblocks-gist"><script src="https://gist.github.com/hajekj/17ab3a7a18b1ad545ff000252dc35451.js?file=654-3.cs"></script><noscript><a href="https://gist.github.com/hajekj/17ab3a7a18b1ad545ff000252dc35451#file-654-3-cs">View this gist on GitHub</a></noscript></div>
<!-- /wp:coblocks/gist -->

<!-- wp:paragraph {"coblocks":[]} -->
<p>And there you go, you can now persist state across authentication requests.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph {"coblocks":[]} -->
<p>Additionally, you should keep on mind that the properties are part of the authentication ticket (therefore stored in the authentication cookie by default) - and you should <a href="https://hajekj.net/2017/03/20/cookie-size-and-cookie-authentication-in-asp-net-core/">beware of its size as I previously explained in an article</a>.</p>
<!-- /wp:paragraph -->