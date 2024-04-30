
!!! tip "Best practise"
    Specify Exact Versions: Whenever possible, specify exact versions of your dependencies in your requirements file. This ensures that the exact same versions are installed across different environments, reducing the chances of compatibility issues.

# Requirement Specification
There are different ways to specify dependencies in the `requirements.txt` file:

## Exact Version
You can specify an exact version of a dependency by using the == operator followed by the version number. For example:

```makefile
package1==1.0.0
```

## Version Range
You can specify a range of acceptable versions using comparison operators (<, <=, >, >=) or hyphens (-). For example:

```makefile
package2>=2.3.0
package3<1.5.0
package4>=1.0.0,<=2.0.0
package5~=1.2.0  # Compatible release operator (approximately equivalent to >=1.2.0, <1.3.0)
```

## Wildcard
You can use a wildcard (*) to indicate that any version of the dependency is acceptable. However, this is generally not recommended for production environments as it can lead to unpredictability. For example:

```makefile
package6==*
```

## URLs and File Paths
You can specify dependencies using URLs or file paths, which can be useful for referencing packages hosted on version control repositories or local directories. For example:

```makefile
git+https://github.com/user/repo.git@branch
file:///path/to/package.tar.gz
```



## Environment Markers
You can use environment markers to specify dependencies that are only installed under certain conditions, such as the Python version or operating system. For example:

```makefile
package8; python_version >= '3.6'
package9; os_name == 'posix'
```

## Extras
Some packages provide extra features that can be installed optionally. You can specify these extras using square brackets ([]). For example:

```makefile
package10[extra1,extra2]
```


# Comments
You can include comments in requirements.txt using the # character. Comments are ignored by pip when installing dependencies. For example:

```makefile
# This is a comment
package7==1.0.0  # This is another comment
```