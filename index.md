# Bijingtons Blog

<div class="posts">
  {% for post in site.posts %}
    <article class="post">
      <h2><a href="{{ post.url }}">{{ post.title }}</a></h2>
      <div>
        <p class="post_date">{{ post.date | date: "%B %e, %Y" }}</p>
      </div>

      <div class="entry">
        {{ post.excerpt }}
      </div>
      
      <small><a href="{{ post.url }}">READ MORE</a></small>

      <p>
        {% for tag in post.tags %}
          <code class="language-plaintext highlighter-rouge">{{tag}}, </code>
        {% endfor %}
      </p>
      
      <hr>
    </article>
  {% endfor %}
</div>
