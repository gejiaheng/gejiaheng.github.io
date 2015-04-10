---
layout: home
---

<div class="index-content me">
    <div class="section">
        <ul class="artical-cate">
            <li><a href="/"><span>Android</span></a></li>
            <li style="text-align:center"><a href="/tech"><span>Tech</span></a></li>
            <li class="on" style="text-align:right"><a href="/me"><span>Me</span></a></li>
        </ul>

        <div class="cate-bar"><span id="cateBar"></span></div>

        <ul class="artical-list">
        {% for post in site.categories.me %}
            <li>
                <h2>
                    <a href="{{ post.url }}">{{ post.title }}</a>
                </h2>
                <div class="title-desc">{{ post.description }}</div>
            </li>
        {% endfor %}
        </ul>
    </div>
    <div class="aside">
    </div>
</div>
