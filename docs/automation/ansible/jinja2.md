- [Mini-course](https://learn.kodekloud.com/user/courses/jinja2-basics-mini-course)
- [Jinja2 original filters](https://jinja.palletsprojects.com/en/stable/)
- [Ansible extended filters](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_filters.html#)


# Jinja2
the **functions** in Jiaja2 are called - **Filters**, here are some common **filters** in different types:

## String manipulation
```jinja2
The name is {{ my_name }} => The name is Bond
The name is {{ my_name | upper }} => The name is BOND
The name is {{ my_name | lower }} => The name is bond
The name is {{ my_name | title }} => The name is Bond
The name is {{ my_name | replace ("Bond", "Bourne") }} => The name is Bourne
The name is {{ first_name | default("James") }} {{ my_name }} => The name is James
Bond
```

## List and set
```jinja2

{{ [ 1, 2, 3 ] | min }}             => 1
{{ [ 1, 2, 3 ] | max }}             => 3
{{ [ 1, 2, 3, 2 ] | unique }}       => 1, 2, 3
{{ [ 1, 2, 3, 4 ] | union( [ 4, 5 ] ) }}        => 1, 2, 3, 4, 5
{{ [ 1, 2, 3, 4 ] | intersect( [ 4, 5 ] ) }}    => 4
{{ 100 | random }}                              => 23
{{ ["The", "name", "is", "Bond"] | join(" ") }} => The name is Bond
```

## Loop
```jinja2
{% for number in [0,1,2,3,4] %}
{{ number }}
{% endfor %}
```

!!! warning 
    `{%` indicates this a part of a **code block**, not a **oneliner**

## Condition
```jinja2
{% for number in [0,1,2,3,4] %}
{% if number == 2 %}
    {{ number }}
{% endif %}
{% endfor %}
```

# Jinja2 in Ansible
**Jinja2** is extendable, Ansible extended it with its own **filters**. ([Ansible Filters](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_filters.html#))

## file
```jinja2
{{ "/etc/hosts" | basename }}               => hosts
{{ "c:\windows\hosts" | win_basename }}     => hosts
{{ "c:\windows\hosts" | win_splitdrive }}   => ["c:", "\windows\hosts"]
{{ "c:\windows\hosts" | win_splitdrive | first }}   => "c:"
{{ "c:\windows\hosts" | win_splitdrive | last }}    => "\windows\hosts"
```


# example
```yaml
# /etc/ansible/hosts
[web_servers]
web1 ansible_host=172.20.1.100
web2 ansible_host=172.20.1.101
web3 ansible_host=172.20.1.102
```


```yaml
# playbook.yml 
-
hosts: web_servers
tasks:
    - name: Copy index.html to remote servers
      template:
        src: index.html.j2
        dest: /var/www/nginx-default/index.html
```

```jinja2
# index.html.j2
<!DOCTYPE html>
<html>
    <body>
        This is a {{ inventory_hostname }} Server
    </body>
</html>
```