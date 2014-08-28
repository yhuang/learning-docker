http://www.snip2code.com/Snippet/63811/Installing-pandoc-on-Mac-OSX-using-Homeb

1.  Generate `learning-docker.md`.

    ```
    Host% cat get-started-with-docker.md running-boot2docker-mac-os-x.md getting-inside-running-docker-containers-without-ssh.md preflight-checks.md running-postgresql-service-docker-container.md running-rails-app-over-two-docker-containers.md docker-container-deep-dive.md building-images-with-dockerfiles.md dockerfile-walkthrough.md linking-dockerized-application-database-mac-os-x-host.md starting-over-mac-os-x.md > learning-docker.md
    ```

2.  Generate `learning-docker.html` from `learning-docker.md`.

3.  Add author `<meta>` tag in `<header>`.

	```
	<meta name="author" content="Jimmy Y. Huang">
	```

4.  Remove these two lines.

	```
	<h2>Learning Docker</h2>

	<h5>Jimmy Y. Huang</h5>
	```

5.  Generate `learning-docker.pdf` from `learning-docker.html`.

	```
	Host% pandoc learning-docker.html \
	-V geometry:margin=1in \
	-V geometry:paperwidth=10in \
	-V geometry:paperheight=16in \
	--latex-engine=xelatex \
	-o learning-docker.pdf
	```