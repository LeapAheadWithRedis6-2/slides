# slides-wip

## Getting Started

1.  Clone the repository
    ```
    git@github.com:LeapAheadWithRedis/slides-wip.git
    ```
2.  CD into the repository
    ```
    cd slides-wip
    ```
3.  Use docker to serve the content
    ```
    docker run --name leap-ahead-with-redis -v $PWD/:/usr/share/nginx/html:ro -d -p 8002:80 nginx
    ``` 
4.  Open browser to http://localhost:8002/