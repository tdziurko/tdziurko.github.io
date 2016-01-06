## Running locally


rake generate   # Generates posts and pages into the public directory
rake watch      # Watches source/ and sass/ for changes and regenerates
rake preview    # Watches, and mounts a webserver at http://localhost:4000


## Publishing blog

rake generate

rake deploy


And then commit

    git add .
    
    git commit -m 'your message'
    
    git push origin source