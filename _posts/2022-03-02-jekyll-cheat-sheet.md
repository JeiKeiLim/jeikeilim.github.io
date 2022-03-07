---
title: "Jekyll Cheat Sheet 모음"
last_modified_at: 2022-03-07T13:59
categories:
  - Blog
tags:
  - Jekyll
  - minimal-mistakes
  - MarkDown
  - CheatSheet
toc: true
toc_sticky: true
---

Jekyll 로 블로그 글을 작성하면서 자주 쓰게 되는 Markdown 문법을 여기에 정리할 계획 입니다.
해당 페이지는 수시로 업데이트 될 예정입니다. Jekyll의 minimal-mistakes 테마를 기준으로 작성되었습니다.

# Image
## Image with caption

```liquid
{% raw %}
{% include figure 
    image_path="assets/images/kindle.jpg" 
    alt="Kindle" 
    caption="This is a caption of the fire image"
%}
{% endraw %}
```
{% include figure 
    image_path="assets/images/kindle.jpg" 
    alt="Kindle" 
    caption="This is a caption of the fire image"
%}

## Image width adjustment

```md
![Stack vertical](/assets/images/kindle.jpg){: width="10%"}
![Stack vertical](/assets/images/kindle.jpg){: width="25%"}
![Stack vertical](/assets/images/kindle.jpg){: width="50%"}
```
![Stack vertical](/assets/images/kindle.jpg){: width="10%"}
![Stack vertical](/assets/images/kindle.jpg){: width="25%"}
![Stack vertical](/assets/images/kindle.jpg){: width="50%"}


# Text
## Raw text with liquid
마지막 % } 의 공백은 붙여줘야 한다.
{% raw %}
```liquid
{% raw %} {% some syntax %} {% endraw % }
```
{% endraw %}

{% raw %} {% some syntax %} {% endraw %}

## Comments
{% raw %}
```liquid
{% comment %}
This is a markdown comment that will not be shown in rendered HTML.
{% endcomment %}
```
{% endraw %}

{% comment %}
This is a markdown comment that will not be shown in rendered HTML.
{% endcomment %}

## Hyperlink with _blank target
{% raw %}
```markdown
[limjk.ai](https://limjk.ai){:target='_blank'}
```
{% endraw %}
[limjk.ai](https://limjk.ai){:target='_blank'}

