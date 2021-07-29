# Bijingtons Blog

<div class="posts">
  {% for post in site.posts %}
    <article class="post">
      <h2><a href="{{ post.url }}">{{ post.title }}</a></h2>

      <div class="entry">
        {{ post.excerpt }}
      </div>
      
      <small class="read_more"><a href="{{ post.url }}">READ MORE</a></small>

      <p>
        <div>
          {% for tag in post.tags %}
            <code class="language-plaintext highlighter-rouge">{{tag}}, </code>
          {% endfor %}
        </div>
      
        <div class="post_date">{{ post.date | date: "%B %e, %Y" }}</div>
      </p>
      

      
      <hr>
    </article>
  {% endfor %}
</div>
