```bash
docker --version | cut -d  ' ' -f3 | tr -d ','
```

- `|` (Pipe)
- `cut` is a command used to split a string into fields based on a specified delimiter and extract specific fields.
    - `-d ''` (Delimiter)
    - `-f3` (Field): -f option specifies which field to extract. -f3 means you want the third field based on the delimiter provided.


- `tr` stands for "translate" and is used to delete or replace characters.
    - `-d` option tells tr to delete specific characters. In this case, -d ',' will delete any commas from the input.




```bash
ansible --version | awk 'NR==1 {print $2}'
```

- `awk` is named after its three creators: Alfred Aho, Peter Weinberger, and Kenneth Thompson. The name itself doesn't directly describe its functionality, but it is a powerful text processing tool that works by dividing lines into fields (by default, fields are separated by spaces or tabs).
    - `NR==1`: NR stands for "Number of Records," which essentially means the line number. In this case, NR==1 means that this rule will apply only to the first line of input.
    - `{print $2}`: This tells awk to print the second field from the first line of input. In the output of docker --version, the fields are separated by spaces
