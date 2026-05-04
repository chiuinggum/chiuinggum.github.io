Visit [chiuinggum.github.io](https://chiuinggum.github.io)

# Local Development Setup

This project uses Ruby, Bundler, and Jekyll.

## One-time Setup

Install `rbenv` and `ruby-build`:

```bash
brew install rbenv ruby-build
```

Enable `rbenv` for zsh:

```bash
echo 'eval "$(rbenv init - zsh)"' >> ~/.zshrc
exec zsh
```

Install and select Ruby:

```bash
cd /chiuinggum.github.io
rbenv install 3.3.6
rbenv local 3.3.6
rbenv rehash
```

Install the Bundler version required by `Gemfile.lock`:

```bash
gem install bundler -v 2.6.6
bundle install
```

## Daily Usage

After opening a new terminal:

```bash
cd /Users/yuntingchiu/Development/chiuinggum.github.io
bundle exec jekyll serve
```

## Check Ruby Environment

Run:

```bash
which ruby
ruby -v
which bundle
```

The Ruby path should look like:

```bash
/Users/yuntingchiu/.rbenv/shims/ruby
```

It should not be:

```bash
/usr/bin/ruby
```

If Ruby still points to `/usr/bin/ruby`, reload zsh:

```bash
exec zsh
```

Then check again:

```bash
which ruby
ruby -v
```
