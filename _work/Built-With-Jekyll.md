---
layout: article
title: "Built With Jekyll"
date: 2014-07-17 06:57:00
cover: /work/img/Built-With-Jekyll-Cover.png
collection: work
tags:
  - all
  - code
  - html
  - css
---

andrewking(me) is hosted from [Jekyll](http://jekyllrb.com) source on [GitHub Pages](http://pages.github.com).

<!--more-->

{% include image.html image="Built-With-Jekyll-001.png" caption="Summary of GitHub Pull Requests, Issues Opened, and Commits" %}

Resources: [Jekyll](http://jekyllrb.com), [GitHub Pages](http://pages.github.com), [kramdown](http://kramdown.gettalong.org), [Sass](http://sass-lang.com), [Pygments](http://pygments.org), [Simple Grid](http://thisisdallas.github.io/Simple-Grid/), [Google Fonts](http://www.google.com/fonts)<br>
Software: [Adobe Brackets](http://brackets.io), [GitHub for Mac](http://mac.github.com), [Pixelmator](http://www.pixelmator.com)

YAML Front Matter and Markdown source of this page:

{% highlight ruby %}
{% raw %}
---
layout: article
title: "Built With Jekyll"
date: 2014-07-17 06:57:00
cover: /img/work/Built-With-Jekyll-Cover.png
section: work
tags:
  - all
  - code
  - html
  - css
---

andrewking(me) is hosted from [Jekyll](http://jekyllrb.com) source on [GitHub Pages](http://pages.github.com).

<!--more-->

{% include image.html image="Built-With-Jekyll-001.png" caption="Summary of GitHub Pull Requests, Issues Opened, and Commits" %}

Resources: [Jekyll](http://jekyllrb.com), [GitHub Pages](http://pages.github.com), [kramdown](http://kramdown.gettalong.org), [Sass](http://sass-lang.com), [Pygments](http://pygments.org), [Simple Grid](http://thisisdallas.github.io/Simple-Grid/), [Google Fonts](http://www.google.com/fonts)<br>
Software: [Adobe Brackets](http://brackets.io), [GitHub for Mac](http://mac.github.com), [Pixelmator](http://www.pixelmator.com)
{% endraw %}
{% endhighlight %}

article.html _layout for Jekyll build:

{% highlight html %}
{% raw %}
<!doctype html>
<html>
{% include head.html %}
<body>
  {% include header.html %}
  
  <section>
    <div class="grid grid-pad">
      <div class="col-4-12">
        <span class="title">{{ page.title }}</span>
        {% if page.date %}
          <p>{{ page.date | date_to_string }}</p>
        {% endif %}
        {% if page.tags %}
          <p>
            {% for tag in page.tags %}
              {% if tag != "all" %}
                <a href="/{{ page.section }}/{{ tag | output }}/">#{{ tag | output }}</a>
              {% endif %}
            {% endfor %}
          </p>  
        {% endif %}
      </div>
      <div class="col-8-12">
        {{ content }}
        {% include article_signoff.html %}
      </div>
    </div>
  </section>

</body>
</html>
{% endraw %}
{% endhighlight %}

Browse the source code at [https://github.com/andrewkingme/andrewkingme.github.io](https://github.com/andrewkingme/andrewkingme.github.io).
