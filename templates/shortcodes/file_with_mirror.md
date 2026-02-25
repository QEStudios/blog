{% if get_env(name="USE_CDN", default="false") == "true" %}
    {% set url = "https://files.skmgeek.com/blog/" ~ path %}
{% else %}
    {% set url = "/files/" ~ path %}
{% endif %}

*[{{ text }}]({{ url }})&nbsp;&nbsp;&nbsp;[(Mirror)]({{ mirror }})*