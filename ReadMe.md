## Wordpress along with Rails app with Apache2 and Passenger

### Intro
This document helps to run Wordpress along with Rails app in a same server and make it available / accessible via Rails App. In this example we want to run a Rails app that accsesible via domain name www.dhanesh-example.com and also want to run a Wordpress instance that accessible via /blog. 

1. When user access the www.dhanesh-example.com or any paths (except /blog /api), it will refer to the Rails app
2. When user access the www.dhanesh-eample.com/blog or any paths inside /blog path scope it will refer to wordpress.


