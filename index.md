<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url | relative_url }}">{{ post.title }} - {{ post.date }}</a>
    </li>
  {% endfor %}
</ul>
