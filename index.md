# Bijingtons Blog

<dl>
  {% for post in site.posts %}
    <dd>
      <h2><a href="{{ post.url }}">{{ post.title }}</a></h2>
      <div>
        <p class="post_date">{{ post.date | date: "%B %e, %Y" }}</p>
      </div>

      <div class="entry">
        {{ post.excerpt }}
      </div>

      <p>
        {% for tag in post.tags %}
          <code class="language-plaintext highlighter-rouge">{{tag}}, </code>
        {% endfor %}
      </p>
    </dd>
  {% endfor %}
</dl>
