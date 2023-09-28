+++
title = 'Hosting My Own Personal Site - Hugo Static Website Generator'
date = 2023-09-25T19:00:00Z
draft = true
+++

Having a personal website with a custom domain to write my thoughts, document my learnings, and put up my projects have always been a long-time goal of mine. I recently decided to follow up through that goal and bought a domain name along with a VPS (Virtual Private Server) subcription to host my website. Some of you probably asking why a VPS if there are more cheaper options? Well because I want to learn on how to host my own website from scratch.

To create this website I researched a lot of options I could use, from the cloud provider to the tech stacks to use. For cloud provider, I wanted to use digitalocean at first due to their "free trial" with limited credits, but unfortunately they suspended my account right after I put my credit card information. From all the infos I gathered in the internet, they have a tendency to block accounts that use new credit card as a fraud prevention. Decided that contacting them would require too much efforts on my part, I chose a local cloud provider in my country, idcloudhost. For the website tech stacks, I got a few options from the recommendation of others on the internet and decided to use Hugo as I only need a static website for this project of mine.

In my first post, I want to document the steps I did to hosting up my website. You can see the tools I use below:
1. Linux based OS for the server
2. Hugo as a site generator
3. PaperMod - Hugo Theme
4. Apache 2 as web server
5. Git and Github

## A little about Hugo
Hugo is a static-website generator. It's fast, simple, free, open source, and support multi-language website. Hugo also have pre-built theme that will help you to create your website as fast as possible. But if you want more customization, you can also create your own template if you know some html, css, and javascript. Overall, it is not too hard to setup and I'll show how I did it.

## Installation
### Installing Hugo
I installed Hugo on my Linux VPS (as a production) and on my local PC (as development). On the VPS, I installed using the prebuilt binaries while I use docker on my local PC.

#### Installing Prerequisites
While these softwares are not required for all case. I decided to install all of them for future usage:
1. Git
2. Go
3. Dart Sass
   
#### Prebuilt Binaries Installation - Linux
1. Go and download Hugo (https://github.com/gohugoio/hugo/tags)
2. Download the version you want to use. For my website I use the version 0.118.2