# Bijingtons Blog

<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a>
      <div markdown="1">
        <p>
          {% for tag in post.tags %}
            `{{tag}}`
          {% endfor %}
        </p>
      </div>
    </li>
  {% endfor %}
</ul>
