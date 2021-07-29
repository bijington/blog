# Bijingtons Blog

<ul>
  {% for post in site.posts %}
    <li>
      <h1><a href="{{ post.url }}">{{ post.title }}</a></h1>
      <div>
        <p class="post_date">{{ post.date | date: "%B %e, %Y" }}</p>
      </div>
      <p>
        {% for tag in post.tags %}
          <code class="language-plaintext highlighter-rouge">{{tag}}, </code>
        {% endfor %}
      </p>
    </li>
  {% endfor %}
</ul>
