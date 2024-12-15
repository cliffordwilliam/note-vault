<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url | relative_url }}">{{ post.title }} - {{ post.date | date: site.date_format }}</a>
    </li>
  {% endfor %}
</ul>
