---
layout: page
title: Posts
tagline: Some side projects
---
{% include JB/setup %}

##### This blog is a place for exploring things that are interesting me at the moment and a record of some of the projects I've worked on recently.

I've gone through a number of these over the years and as of 2017 this is the most recent iteration, replacing those that came before.

<div class="table-responsive">
  <table class="table table-striped">
    <tr><th>Post</th><th>Date</th><th>Category</th><th style="width:40%">Tags</th></tr>
    {% for post in site.posts %}
      <tr>
        <td><a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></td>
        <td>{{ post.date | date_to_string }}</td>
        <td><a href="{{ BASE_PATH }}/categories/{{ post.category }}#{{ category[0] }}-ref">{{ post.category }}</a></td>
        <td><ul class="tag_box">
          {% assign tags_list = post.tags %}
          {% include JB/tags_list %}
        </ul></td>
      </tr>
    {% endfor %}
  </table>
</div>
