nignx.conf中的
```conf
http {
	...
	server {
		listen 3000;
	}
}
```

其中的3000应该与Dockerfile中expose的port是一样的！比如:
```Dockerfile
...

EXPOSE 3000
...
```