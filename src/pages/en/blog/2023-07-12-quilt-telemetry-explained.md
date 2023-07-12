---
title: "Quilt Telemetry Explained"
date: 2023-07-12 12:25:00 -00:00
authors:
  - Southpaw1496
layout: /src/layouts/Post.astro
---
Quilt Loader 0.19.2 adds beacon functionality to count Quilt's Monthly Active Users, or MAUs. There has recently been some confusion about how this functionality works and why it was added, so this blog post is intended to provide a clear explanation of both of those things.

<!-- MORE -->

## How the telemetry works
Most telemetry is gathered with the intention of precisely learning how different demographics of people are using a product, perhaps with the intention of analysing behaviour to fine-tune an algorithm, interface, or marketing campaign. The word "telemetry" is associated with all sorts of sensitive data: your IP address, your location, the intimate details of how you're using an app, but we're not collecting anything like that. In Quilt's case, we only need to know how many people are using Quilt in the wild, so we optimised for sharing as little data as we possibly could while still accomplishing this goal.

When Quilt Loader is launched, it first checks if the user has opted out of telemetry (more on this later), or if it has already added you to this month's active user count. If it neither of those apply, it sends an empty request to `beacon.quiltmc.org`, containing no data except the bare minimum that it needs to establish a connection. No identification, no information about how you're using Quilt Loader, no nothing. It then records the fact that it has made this request in a local file that is shared among all your Minecraft instances, so that it doesn't send any more requests that month. After receiving the request, the Beacon server increments the Monthly Active User counter, then deletes all the connection data associated with the request that it just received.

![A cute graph explaining the process Quilt Loader goes through to update the MAU beacon.](/assets/img/writing/blog/2023-06-26-mau-beacon/beacon-update-process.png)
Image Credit: ixorai

It's important to emphasise just how little data is being sent here. You're sending more data away if you:
- Play Minecraft itself.
- Use CurseForge, Modrinth, or any mods or launchers that integrate with them (such as Prism or ModMenu).
- Connect to any website on the internet.

If you want to dig into the exact workings of the system, you can look at [Quilt Loader PR #326](https://github.com/QuiltMC/quilt-loader/pull/326), [RFC 81](https://github.com/QuiltMC/rfcs/blob/main/specification/0081-active-user-beacon.md) which outlines the functionality of the active user beacon, and the [Beacon server itself](https://github.com/QuiltMC/beacon.quiltmc.org), which is fully open-source.

## Why are we adding telemetry?
Monthly Active Users is a statistic used all over the technology industry, often as a way to assess a platform's size or value. Although we've made sure that the telemetry we've added is as privacy-preserving as possible, it wouldn't be worth the disadvantages, however slight, if we didn't actually need it for some reason.

The main reason we need a Monthly Active User count is that many companies and organisations simply won't work with us if they don't consider us large or consequential enough to be worth their time. Most notably, CurseForge will only consider adding Quilt to their launcher and modpack system if we can give them a Monthly Active User count to prove that Quilt is large enough to justify the development effort. Additionally, although this hasn't happened yet, knowing our Monthly Active User count gives us a widely-accepted measure of size to use when applying for any sponsorships, grants, or other partnerships that we may need in the future.

## But Why Couldn't You Just...
We did consider some alternatives, however, everything we came up with was unsuitable for various reasons:
- Measuring QSL downloads is inaccurate, because everyone downloads a new version of QSL whenever it updates.
- Measuring Installer downloads ignores a large portion of users who use 3rd-party launchers, like Prism.
- Measuring downloads of the Loader itself from Maven is technically infeasible because it is downloaded directly from a cloud storage bucket, and there's no point where you can reasonably insert a counter for download requests without spending lots of money.

Additionally, all of those methods simply measure download count, which, while a useful statistic, is not as universally accepted as a Monthly Active User count.

## How to disable the telemetry
If, despite the precautions we've taken to measure telemetry in a privacy-preserving way, you'd still like to disable it, there are a couple of ways that you can do so.
1. Set the environment variables `QUILT_DISABLE_BEACON` or `CI` to `true`. This can be accomplished by opening the `~/.bashrc` file in Linux or `~/.zshrc` file in macOS (simply type `open ~/.bashrc` or `open ~/.zshrc` in the Terminal to open them in the default text editor) and adding the line `export QUILT_DISABLE_BEACON=true`.
2. Add the Java argument `-Dloader.disable_beacon=true`. In the vanilla launcher, you can add Java arguments in Installations -> Edit -> More Options -> JVM Arguments. Note that this will only disable the telemetry for the Installation that you add the Java argument to.