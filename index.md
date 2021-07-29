# Bijingtons Blog

<ul>
   {% for post in site.posts %}
     <li>
       <a href="{{ post.url }}">{{ post.title }}</a>
       <p>
        {% for tag in post.tags %}
          `{{tag}}`
        {% endfor %}
       </p>
     </li>
   {% endfor %}
 </ul>
