---
layout: post
title: Keeping Exoteric Jobs on GitHub Actions Fast
categories: [ci, github actions]
excerpt: GitHub Actions was a great addition to GitHub. They streamline the development workflow by keeping all continuous integrations and deployment processes inside the project's repository instead of in an external tool.
last_modified_at:
---

2.  [Simple Builds](#orgee6948d)
3.  [Caching libs](#org1cb0da9)
4.  [What if I need to install some custom libs?](#org8b6092c)
5.  [What if I decide to add docker?](#org1d1f6ef)
6.  [What if I want to build and run something on docker?](#orgdca10a5)


GitHub Actions was a great addition to GitHub. They streamline the development workflow by keeping all continuous integrations and deployment processes inside the project's repository instead of in an external tool. Developers can now build, test, and deploy projects on Linux, macOS, and Windows platforms within their GitHub repository. As the usual CI/CD platforms GitHub Actions also supports:

-   Containers or virtual machines
-   Multiple languages
-   Multi-container apps

I have made some experiments in small projects with no complaints, it's clean, easy to set up, and practical. However, I have also tested for bigger projects with complex build requirements. My use case was:

-   I have a python project
-   I have a test suit to run
-   I have a **specific** list of requirements (including python and C libraries)
-   I need a database
-   I want tests to run in every commit
-   I want tests to run fast

Let's go through how GitHub Actions goes about satisfying these requirements for complex builds.


<a id="orgee6948d"></a>

# Simple Builds

Simple builds on Github Actions are sweet. In this post, you will not find details on how simple builds work, so if you are new to CI/CD, or to what are actions the links below will help you foster an understanding of the topic:

-   Continuous Integration: <https://www.martinfowler.com/articles/continuousIntegration.html>
-   Continuous Deployment: <https://www.atlassian.com/continuous-delivery/continuous-deployment>
-   GitHub Actions for Python projects: <https://dan.yeaw.me/posts/github-actions-automate-your-python-development-workflow>

This post will extend on a simple build described below:

```yml
name: Build

on:
  push:
    branches:
      - "\*"
      - "****/****"

jobs:
  test-linux:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: [3.8.6]
    steps:
      - uses: actions/checkout@v2
      - name: Set up Python ${{ matrix.python-version }}
        uses: actions/setup-python@v2
        with:
          python-version: ${{ matrix.python-version }}
      - name: Install dependencies
        run: |
          pip install -r test-requirements.txt
      - run: |
        python3 -m pytest

```

All it does is to run tests on every push for every branch. Note that this is as simple as it can be.


<a id="org1cb0da9"></a>

# Caching libs

The first improvement one can make is to cache dependencies, since they don't change very often we can save some time during builds just by having them stored for later usage.

```yml

  - name: Cache install test-requirements
    uses: actions/cache@master
    id: cache-pip
    with:
      path: ~/.cache/pip
      key: ${{ runner.os }}-pip-${{ hash }}
      restore-keys: |
        ${{ runner.os }}-pip-

```

A few comments about how this works on Github Actions:

-   It searches for a saved cache based on a key you pass
-   If the actions don't find a cache, the requirements are installed
-   The job proceeds to run all the steps
-   If everything passes the cache is saved for a next run

There are two details here. When using ` ~/.cache/pip` as the cache path the job is not caching the library installation, what it is doing is making use of the pip cache and merely caching the wheels that allow us to install the lib. This means that the cache is only avoiding a trip to the remote register while in runtime is still installing the libs.

One different approach is to cache the whole installed packages and by proxy the complete installation. In our case:

```yml

  - name: Cache install test-requirements
    uses: actions/cache@master
    id: cache-pip
    with:
      path: /opt/hostedtoolcache/Python/3.8.6/x64/lib/python3.8/site-packages/
      key: ${{ runner.os }}-pip-${{ hash }}
      restore-keys: |
        ${{ runner.os }}-pip-

```
\*OBS: The path changes for each platform<sup><a id="fnr.1" class="footref" href="#fn.1">1</a></sup>

My simple build using only pip cache took 14s to install all dependencies. After changing the cache to be the packages folder, the procedure now takes 3s to restore the cache and 3s for installing the libraries (outputting `Requirement already satisfied:` for all libraries).

Even though the speed improvement is substantial, this approach is still fragile<sup><a id="fnr.2" class="footref" href="#fn.2">2</a></sup> because it is sensitive to paths and OS versions. Another issue is that this doesn't scale well, being that as your requirements grow this type of cache will get slower to make, to the point that you will have to weigh which one is faster: saving the cache and then restoring it, or just using the regular pip cache. In my real-life example caching the whole `site-packages` folder took something around 20 min while the installation using pip cache only took 50s.


<a id="org8b6092c"></a>

# What if I need to install some custom libs?

What happens if you need to compile and build a lib on every job to run your tests? The answer is that you get a major slow down on build times. Github Actions cache allows us to extend the cache functionality by keeping the compiled files and then speed up the builds. As an example of such a library, I will use libpostal<sup><a id="fnr.3" class="footref" href="#fn.3">3</a></sup> a C library for parsing street addresses around the world:

```yml

  - name: Cache libpostal
    uses: actions/cache@v2
    id: libpostal-cache
    with:
      path: ${{ env.CACHE_DIR }}     # custom variable
      key: ${{ runner.os }}-libpostal-cache
  - name: Install libpostal
      if: steps.libpostal-cache.outputs.cache-hit != 'true'
      run: |
        sudo apt-get update
        sudo apt-get install curl autoconf automake libtool pkg-config
        mkdir -p ${{ env.CACHE_DIR }}
        rm -rf ${{ env.CACHE_DIR }}/libpostal
        cd ${{ env.CACHE_DIR }}
        git clone https://github.com/openvenues/libpostal
        cd libpostal
        ./bootstrap.sh
        ./configure --datadir=${{ env.CACHE_DIR }}
        make -j4
        sudo make install
        sudo ldconfig

  - name: Install test-requirements
    run: |
      cd ${{ env.CACHE_DIR }}/libpostal
      sudo make install
      sudo ldconfig ${{ env.CACHE_DIR }}
      cd $PROJECT_ROOT   # custom variable
      pip install -r test-requirements.txt

```
\*OBS: CACHE_DIR and PROJECT_ROOT are custom variables only necessary to ease the visualization.

Github Actions make available the cache hits historic for each step of the pipeline allowing us to successfully skip some steps when the content is recovered from the cache. This feature allows us to separate the slow part from the quick part of building libpostal, the slow part being the compiling part. By caching the compilation result that lives inside the libpostal folder, we are always restoring the files allowing the installation to resume itself with `sudo make install && sudo ldconfig` which runs pretty quickly.


<a id="org1d1f6ef"></a>

# What if I decide to add docker?

Docker is extremely useful in most situations, in this case, I'm using it to set a database for the tests. Docker Compose is a great tool to spin these on-demand containers, we can use it directly by adding:

```yml

  - run: |
    docker-compose up --detach postgres
    python3 -m pytest
    docker-compose down

```

For the code above to work you will need a docker-compose.yml file with a Postgres service specification like this:

```yml
Postgres:
  image: postgres:latest
  ports:
    - "5432:5432"

```


<a id="orgdca10a5"></a>

# What if I want to build and run something on docker?

Building and running images are a good fit for eventual jobs, for frequent builds (such as tests suits) this may turn out to be a major slow down. Depending on your Dockerfile builds may take a long time besides just the docker setup time already makes docker jobs slower than local jobs (if you are running tests for example). One speedup for complex setups is to pull from Dockerhub or ECR  a built image, however, depending on the size of your image you will also have to cope with the time to pull on every run.

If you must build and run containers on-demand you will need to cope with no-cache since Docker Layer Caching is not natively integrated with GitHub Actions, some workarounds can be made but they are case-specific<sup><a id="fnr.4" class="footref" href="#fn.4">4</a></sup>. Some actions on the marketplace claim to give docker layer cache for GitHub Actions but they are not reliable for larger projects will frequent builds.

---

In this post, we discussed some ways to keep complex builds fast by using GitHub Actions features. We discussed cache-specific python libraries, cache on compiled libraries, multi-container apps, and on-demand docker builds and runs. Github Actions is a great tool equivalent to other CI/CD tools on the market, except for Docker Layer Caching which is not natively supported.

# References
- <https://dan.yeaw.me/posts/github-actions-automate-your-python-development-workflow/>

# Footnotes

<sup><a id="fn.1" href="#fnr.1">1</a></sup> <https://docs.github.com/en/free-pro-team@latest/actions/guides/building-and-testing-python#specifying-a-python-version>

<sup><a id="fn.2" href="#fnr.2">2</a></sup> <https://github.com/actions/cache/issues/175#issuecomment-636799040>

<sup><a id="fn.3" href="#fnr.3">3</a></sup> <https://github.com/openvenues/libpostal>

<sup><a id="fn.4" href="#fnr.4">4</a></sup> <https://github.com/actions/cache/issues/31>

