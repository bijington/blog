# Bijingtons Blog

<ul>
   {% for post in site.posts %}
     <li>
       <a href="{{ post.url }}">{{ post.title }}</a>
       {% for keyword in post.keywords %}
        `{{keyword}}`
       {% endfor %}
     </li>
   {% endfor %}
 </ul>
