# Bijingtons Blog

<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a>
        <p>
          {% for tag in post.tags %}
            <code class="language-plaintext highlighter-rouge">{{tag}}</code>
          {% endfor %}
        </p>
    </li>
  {% endfor %}
</ul>
