# Bijingtons Blog

<dl>
  {% for post in site.posts %}
    <dd>
      <h4><a href="{{ post.url }}">{{ post.title }}</a></h4>
      <div>
        <p class="post_date">{{ post.date | date: "%B %e, %Y" }}</p>
      </div>
      <p>
        {% for tag in post.tags %}
          <code class="language-plaintext highlighter-rouge">{{tag}}, </code>
        {% endfor %}
      </p>
    </dd>
  {% endfor %}
</dl>
