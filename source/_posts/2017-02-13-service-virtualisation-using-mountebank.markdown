---
author: Aby George A
comments: true
date: 2017-02-13 11:13:58+00:00
excerpt: Service virtualisation using mountebank
layout: post
link: /blog/2017/02/13/service-virtualisation-using-mountebank/
slug: service-virtualisation-using-mountebank
title: Service Virtualisation Using Mountebank
wordpress_id: 318
keywords: Mountebank , Stub , Service Virtualisation 
description: How to get started using Mountebank
categories:
- Mountebank
- Service Virtualisation
tags:
- Stubs
---

# What is Mountebank?




<blockquote>As per mbtest.org "_mountebank is the first open source tool to provide cross-platform, multi-protocol test doubles over the wire. Simply point your application under test to mountebank instead of the real dependency, and test like you would with traditional stubs and mocks_"</blockquote>


In short mountebank is a open source service virtualisation tool . Mountebank uses imposters to act as on demand test doubles. Hence our test cases communicate to Mountebank and mountebank responds back with relevant stubs as defined.


# How to Setup Mountebank ?


Installation can be done via two methods


### npm


Mountebank can be installed as a npm package. Node.js should be installed for this option to work

    
    <code>npm install -g mountebank</code>




### Self contained Installation file


OS Specific installation file can be downloaded from [Download](http://www.mbtest.org/docs/install)

Note: Please read through the windows path limitation mentioned in above link
