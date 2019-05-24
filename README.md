# Wraith with Chrome
A responsive screenshot comparison tool

## Cloning and installing

Clone wraith

```
cd /Users/[path-to-where-wraith-will-be-installed]
```

```
git clone https://github.com/cschnell/wraith.git
```

```
cd wraith
```

Create Docker Image with included Dockerfile:

```
docker build -t cs/wraith .
```

=> cs/wraith is just a namespace, you can change it if you want but all further cli commands have this namespace

The container should run now but if it doesn't start it with

```
docker run cs/wraith
```

Create a shell alias to wraith config files with the namespace of the container
You might want to make the alias permanent (see https://coolestguidesontheplanet.com/make-an-alias-in-bash-shell-in-os-x-terminal/)

```
alias docker-wraith="docker run --rm -P --shm-size=512m -v /Users/<path-to-where-wrait-is-installed>/wraith:/wraithy -w='/wraithy' cs/wraith"
```

If running into errors when using wraith consider raising the --shm-size option to 1G

## Updating the image

It might be required to update the image from time to time. For that you have to remove the shots and shots_base directories from the wraith folder
```
mv shots ../
mv shots_base ../
```

Then build the image with the `--no-cache` option

```
docker build --no-cache -t cs/wraith .
```

And move the shots directories back to the wraith folder

```
mv ../shots ./
mv ../shots_base ./
```

## Configuring and running

Edit configfiles or create new ones. There is a default example in the folder configs/default

Use a folder structure like this:

`configs/[customer]/[domain]`

### Spider for URL-Crawling

```
docker-wraith spider configs/[customer]/[domain]/capture.yaml
```

### Capture Images in Compare Mode

Run wraith with

```
docker-wraith capture configs/[customer]/[domain]/capture.yaml
```


### Capture Images in History Mode

To capture the base to compare against

```
docker-wraith history configs/[customer]/[domain]/history.yaml
```

To capture the current version and compare it against an existing base and create the gallery

```
docker-wraith latest configs/[customer]/[domain]/history.yaml
```


## Interpretation of results
Wraith stores the images and a gallery at 
`/Users/[path-to-where-wrait-is-installed]/wraith/shots/[customer]/[domain]/`
if configured properly.
The gallery can be viewed in the browser with the url 
`file:///Users//[path-to-where-wrait-is-installed]/wraith/shots/[customer]/[domain]/gallery.html`

## Using Megapages or anchor links
Unfortunately, Wraith spider and Chrome do not support anchor links like http://www.mysite.de/page.html#jumplink
If your page features those you have to make some work-arounds:

1. Spider as usually `docker-wraith spider configs/[customer]/[domain]/capture.yaml`
2. open your spider_paths.yaml
3. in the first section (the path of the images on your disk) remove the % character
4. in the second section (the paths of the page) change %23 to #
5. in capture.yaml switch the  browser from chrome to phantomjs

Now, crawling should work

## Things to remember and tipps
* Always precalculate how many images you are going to produce
* There will be as many images x resolutions as you define domains for comparison
* on a decent Macbook, about 15-30 screens per minute are created, depending on the server.
* The default rendering engine is Headless Chrome. 
  * This is the slowest engine but produces best results in terms of rendering
  * When creating too many images (more than 300 in a row), chrome might run into an increasing number of timeouts
* If you need more performance, change the rendering to `phantomjs`. Phantomhjs can handle an average of 100 images per minute. However, the rendering is not as accurate as headless chrome.
* After spidering review the spider_paths.yml. 
  * It might contain a lot of pages we don't want to test.
  * Be clever.
  * If one news article looks fine, chances are great, others do likewise.
* Make use of the regex url exclusion when spidering. It can save you a lot of time.
* Wraith comes with a cli that can do a lot more (like history). Read the documentation at http://bbc-news.github.io/wraith/

### A simple calculation:
* three Domains
  * dev
  * test
  * live
* three resolutions
  * 480
  * 1024
  * 1280
* 150 pages

This equals a total of 1350 Screenshots. The average is 15-30 Screenshots per minute which results in at least 45-90 minutes until results and gallery are created.
Phantomjs should do this in about 10-15 minutes

## Further Documentation
http://bbc-news.github.io/wraith/
