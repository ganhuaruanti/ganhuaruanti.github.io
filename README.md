# 幹話聊軟體

## Contribute Guide

### Development

**Requirement**:

- ruby: `^2.6.4`
- bundler: `2.0.2`

**Serve Development**:

```sh
# install once
gem install bundler:2.0.2
bundle install

# start the jekyll development
bundle exec jekyll serve

# developing with non-localhost connection like docker container
bundle exec jekyll serve --host 0.0.0.0
```
