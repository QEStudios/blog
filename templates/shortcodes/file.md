{% if get_env(name="USE_CDN", default="false") == "true" %}
    {{ "https://files.skmgeek.com/blog/" ~ path }}
{% else %}
    {{ "/files/" ~ path }}
{% endif %}