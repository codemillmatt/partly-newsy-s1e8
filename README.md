Welcome back to Partly Cloudy! The show where you learn how to build a cloud-connected Xamarin mobile application. We start from nothing and don't quit until it's ready for the App Store!

So go [download all the code for this episode on GitHub](https://aka.ms/partly-cloudy-s1e7-github)! (And if you haven't, [sign up for a free Azure account](https://azure.microsoft.com/free/?WT.mc_id=mobile-0000-masoucou), you'll need it to follow along.)

![Title Image](https://res.cloudinary.com/code-mill-technologies-inc/image/upload/c_scale,e_shadow:40,h_750/v1579107007/thumbnail_25731_sf6f5o.jpg)

# Episode 8 Recap - Mostly Cloudy (Azure CDN + Front Door)

When you create an Azure resource, you specify which data center you want it to reside in. It's geographically "locked", so to speak.

When you create a mobile app, it's meant to be used anywhere in the world. I mean, mobile is right there in the name.

That can lead to perceived performance issues if your cloud resources are on the other side of the world from where your mobile app is being used. And that's what this episode of Partly Cloudy is out to solve - making your mobile app faster by bringing the cloud closer to it.

## What Are We Going To Do?

The whole point of this episode is to get our Azure resources closer to whereever our users are. So everything we're going to do today is going to be in the cloud, or Azure.

The first thing you'll learn about is how to speed up delivery of any static resources, like images, that you don't want to store within your application bundle using Azure BLOB Storage and Content Delivery Network (CDN).

Next you'll see how you can drastically improve the read and write time of any database operation with Cosmos DB by enabling multiple read & write regions.

Then, if you're accessing Cosmos from a Function, those read and write regions won't do any good if your Function isn't as close to the user _and_ the region as possible - so we'll show how Azure Front Door can speed that up too!

You're going to need an Azure account to follow along - if you don't have one, sign up for a [free one here](https://aka.ms/partlycloudyazurefree).

Buckle up!

### Picking Up From Last Time

The app is the exact same as it was from the last episode. No code changes at all to start out with. Everything we're doing today, with the exception of changing some URLs is happening in Azure.

## Azure BLOB Storage and CDN

[Azure BLOB Storage](https://docs.microsoft.com/azure/storage/blobs/storage-blobs-introduction?WT.mc_id=mobile-0000-masoucou) is a great way to store any static - or nonchanging - content for your app that you don't want to include as part of your app bundle. This could include images, videos, etc.

BLOB Storage is cheap, redundant - meaning it's backed up so you don't have to worry about losing any data, and it's fast to get data out of.

In the app, I'm using BLOB storage to host all the images found on the Favorites page.

![favorite screen in app](https://res.cloudinary.com/code-mill-technologies-inc/image/upload/c_scale,e_shadow:40,h_800/v1579105244/Screenshot_1579092257_v0rvvi.png)

But because an Azure Storage account is created in a specific region, and the user's of our app may be far away from that region, it would be nice to have all of those images available to them as quickly as possible.

And that's where a [Content Delivery Network, or CDN](https://docs.microsoft.com/azure/cdn/cdn-overview?WT.mc_id=mobile-0000-masoucou) comes in.

An [Azure CDN can sit on top of an Azure BLOB Storage account](https://docs.microsoft.com/azure/cdn/cdn-create-a-storage-account-with-cdn?WT.mc_id=mobile-0000-masoucou) and cache all of the BLOBs within it. The caching then makes, in our case, the images available "at the edge". The edge means that they're ready to be served up from various Azure data centers far away from the originating one that has the storage account in it.

You can configure a CDN to obey all sorts of different caching rules and standards - like Akamai, Verizon, or Microsoft. All that info and more can be found in the docs.

### Getting the new URLs for the images

Once you get the CDN setup. You'll be brought to the page that has the CDN's endpoint URL.

Then to get to the images in BLOB storage - append the container name, then the image name to it.

So in my case, I have all the images in a `thumbnails` container.

A fully qualified URL then is: `https://partlynewsycdn.azureedge.net/thumbnails/africa.jpg`

### Learn more!

And of course, there's a great [Microsoft Learn module](https://docs.microsoft.com/learn/modules/create-cdn-static-resources-blob-storage/?WT.mc_id=mobile-0000-masoucou) that covers the ins and outs of having a BLOB Storage account backed by a CDN too! Definitely worth checking out!

And if you want to dig deep on everything Azure has for data storage - there's a [Learn module](https://docs.microsoft.com/learn/paths/store-data-in-azure/?WT.mc_id=mobile-0000-masoucou) for that too!

## Azure Cosmos DB Multi-Read and Write

The images are downloading nice and fast! Next up is to get all the data to be saved fast too!

We can do that by enabling multi-read and write regions in Azure Cosmos DB.

You can [read more about what global distribution](https://docs.microsoft.com/azure/cosmos-db/distribute-data-globally?WT.mc_id=mobile-0000-masoucou) means to Cosmos DB here.

But enabling it is pretty simple.

Open up your Cosmos instance in the Azure portal. Go to the `Replicate Data Globally` tab on the left hand side. Then start clicking data regions that you want your data to be replicated to.

![setting up Azure Cosmos DB multi-read and write](https://res.cloudinary.com/code-mill-technologies-inc/image/upload/c_scale,e_shadow:40,h_800/v1579105243/multi-read-cosmos_yesnaq.png)

That's it!

## Knocking on Azure's Front Door

Having multiple read and write regions setup in Azure Cosmos DB is all well and good, but it doesn't do you any good if you're not accessing them directly.

In other words - imagine you have an Azure Function that reads from a Cosmos DB instance.

If the Function is created in the WestUS region, it doesn't do you any good if you have a read region of Cosmos setup in Europe - because the first request to the Function always has to travel to WestUS in order to initiate anything.

That's where [Azure Front Door comes in](https://docs.microsoft.com/azure/frontdoor/front-door-overview?WT.mc_id=mobile-0000-masoucou).

Front Door allows you to setup routing rules to various backend pools, so requests can be routed to the one that will respond the quickest.

> A backend pool is equivalent to a Function App. It's the _backend_ serving requests. You _will_ need to create a backend in each region you want it to serve requests from. So maybe one in the US and one in South America if you think that's where most of your users will come from.

In addition it does so much more!

You can setup Web Application Firewall (WAF) rules to apply filtering to traffic - like geo-filtering. 

And it does automatic failover - so if one of your backend pools goes down, Front Door will automatically route requests to others that it knows are working.

And it does more too! Read about it all here.

> It should be noted that in our app as it stands, we are not using Front Door to route any requests to Azure Cosmos DB because we're using Azure App Center. If you weren't using App Center - or were using Functions or an App Service to hit Cosmos, this is the way to go.

## Summary

In this episode you learned a couple of methods to speed up the response time of Azure so it's fast no matter where in the world your users are.

First off a CDN was used to cache any static resources "on the edge" to avoid any unnecessary network hops while retrieving them from your Azure Storage account.

Then you enabled multi-read and write regions in Azure Cosmos DB so those are blazing fast.

Finally you saw how Azure Front Door can route requests to backend pools that will respond the quickest to the user, no matter where they happen to be. And it provides automatic failover too - so the backend of your app will never be down.

In the next episode of Partly Cloudy - we'll tune the UI of the app so it looks exactly how we want - and then it will be ready for deployment!