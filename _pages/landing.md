---
layout: page
title: Landing
permalink: /landing
comments: false
---
    
<!-- Sticky - add sticky tag to the post you want to highlight here - tags: [sticky] -->
{% for post in site.posts %} 
{% if post.tags contains "sticky" %}
<div class="jumbotron jumbotron-fluid jumbotron-home pt-0 pb-0 mt-3 mb-2rem bg-lightblue position-relative">
    <div class="pl-4 pr-0 h-100 tofront">
        <div class="row justify-content-between">
            <div class="col-md-6 pt-6 pb-6 pr-lg-4 align-self-center">
                <h1 class="mb-3">{{post.title}}</h1>
                <p class="mb-3 lead">
                    {{ post.excerpt | strip_html | strip_newlines | truncate: 136 }}
                </p>
                <a href="{{site.baseurl}}{{post.url}}" class="btn btn-dark">Read More</a>
            </div>
            <div class="col-md-6 d-none d-md-block pr-0" style="background-size:cover;background-image:url({{site.baseurl}}/{{ post.image }});">	
            </div>
        </div>
    </div>
</div> 
{% endif %}
{% endfor %}


    


{% endif %} <!--endif page url is / -->