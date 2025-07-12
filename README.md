# Docker-cgwire

Docker compose for [Kitsu](https://kitsu.cg-wire.com/) and [Zou](https://zou.cg-wire.com/)

## Notice

this project is a detached version of [MathieuBOUZARD's](https://gitlab.com/mathbou) cgwire automation project on [gitlab](https://gitlab.com/mathbou/docker-cgwire) so for make sure to
go through that project as well

## Changes for eefafx

We are using the same flow and same docker conainsers execept some minor
changes to make the process of deployement easier:

- reading the env var from .env
- sourcing the .env file at the start
- added a .env.sample to follow along easily

## Running the compose

for running the compose:

```
chmod +x build.sh
./build.sh local
```
