### Steps taken to generate the requirements for installation

1. Install https://github.com/jazzband/pip-tools
2. Create `requirements.in` and list the requirements as given. You will need `cython` to install `pomegranate` and hence it's listed there. Also, don't forget to remove `pomegranate` from `requirements.txt` after it's generated, since we have explicitly added it to Dockerfile for installation.
3. Run `pip-compile` tool to generate `requirements.txt`. This will pin the specific dependencies needed to be installed and hence will result in reproducible builds.


### How to build the docker image and run the container
1. In the current directory, run `docker build -t my-python-app .`
2. After that run `docker run -it my-python-app`. You will see the version number of the `pomegranate` package as output, which is `0.12.2`.