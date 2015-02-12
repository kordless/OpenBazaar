# Sample Project
Edit this file to show additional information to users launching your project's instances.

The following template variables are available for use in the files in this directory:

    {{ project_name }}
    {{ project_url }}
    {{ donation_address }}
    {{ ipv4_address }}
    {{ ipv6_address }}

You can also toggle sections of text by using the following control syntax:

    {% if ipv4_address %}The IP address of the box is: {{ ipv4_address }}{% endif %}
    
The software also supports a full set of [Markdown syntax](http://daringfireball.net/projects/markdown/syntax) for the README.md file.
