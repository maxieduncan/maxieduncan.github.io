---
layout: post
category : solarnetwork
title: "Sharing Eclipse Code Templates and Formatting"
tags : [solarnetwork, eclipse]
---
{% include JB/setup %}

The [Eclipse IDE](eclipse.org) provides a nice interface via the preferences to import, edit and export Java Code Templates and Formatting rules. If you want to automatically import these settings into your workspace though things get a little more complicated. You can configure project specific settings and add them to the version control of each project but if you want a workspace specific configuration to apply to all your projects it's not as straightforward. With SolarNetwork we have a single workspace with over 200 projects so maintaining them individually in version control isn't practical. Having to manually import the workspace settings isn't exactly a strenuous activity but automating the process and ensuring we have consistent styling is definitely nice.

Fortunately almost everything in Eclipse is managed in plugin configuration and with a bit of digging you can identify the files in question and what properties need to be set. The [gist](https://gist.github.com/maxieduncan/db6f3c7aef1baeafe78fca691c3a6926) below shows some of these configuration options, including the ones that load the XML files that contain your code templates and formatting rules.

<script src="https://gist.github.com/maxieduncan/231d441eb5d4c0a0f0bbefb6c40b7b24.js"></script>

I've worked on a project before which used gradle for the build system and we had a much more robust and easier to maintain plugin to  manage this configuration, especially in regards to the XML management. In SolarNetwork we're relying on shell scripting currently and while the snippet above works for our current configuration, I'm not as confident in it's ability to handle edge cases.

The above getProperty() function is based on [this gist](https://gist.github.com/kongchen/6748525).
