# How to build

## Dependencies

```
sudo apt install jekyll
bundle install
```

To update Gem if you need to:
```
gem update --system
```


## Build

On my buntu 20.04,

`jekyll serve` works

but I think it should be `bundle exec jekyll serve`


## Gotchas
So  I was getting
```
Traceback (most recent call last):
	12: from /usr/local/bin/jekyll:23:in `<main>'
	11: from /usr/local/bin/jekyll:23:in `load'
	10: from /var/lib/gems/2.7.0/gems/jekyll-4.2.2/exe/jekyll:8:in `<top (required)>'
	 9: from /var/lib/gems/2.7.0/gems/jekyll-4.2.2/exe/jekyll:8:in `require'
	 8: from /var/lib/gems/2.7.0/gems/jekyll-4.2.2/lib/jekyll.rb:191:in `<top (required)>'
	 7: from /var/lib/gems/2.7.0/gems/jekyll-4.2.2/lib/jekyll.rb:12:in `require_all'
	 6: from /var/lib/gems/2.7.0/gems/jekyll-4.2.2/lib/jekyll.rb:12:in `each'
	 5: from /var/lib/gems/2.7.0/gems/jekyll-4.2.2/lib/jekyll.rb:13:in `block in require_all'
	 4: from /var/lib/gems/2.7.0/gems/jekyll-4.2.2/lib/jekyll.rb:13:in `require'
	 3: from /var/lib/gems/2.7.0/gems/jekyll-4.2.2/lib/jekyll/drops/collection_drop.rb:3:in `<top (required)>'
	 2: from /var/lib/gems/2.7.0/gems/jekyll-4.2.2/lib/jekyll/drops/collection_drop.rb:4:in `<module:Jekyll>'
	 1: from /var/lib/gems/2.7.0/gems/jekyll-4.2.2/lib/jekyll/drops/collection_drop.rb:5:in `<module:Drops>'
/var/lib/gems/2.7.0/gems/jekyll-4.2.2/lib/jekyll/drops/collection_drop.rb:10:in `<class:CollectionDrop>': undefined method `delegate_method_as' for Jekyll::Drops::CollectionDrop:Class (NoMethodError)
Did you mean?  DelegateClass
```
I tried switching to Jekyll 4.3.2, which didn't fix it, still the same thing.


Ooh okay it was https://stackoverflow.com/questions/68220028/undefined-method-delegate-method-as-for-jekylldropscollectiondropclass-n

So I think the real problem was that I had Apt's jekyll, which was different than Gem's jekyll. Okay.

https://talk.jekyllrb.com/t/new-install-jekyll-4-0-0-on-ubuntu-19-04-gives-error/3262/5
sudo mv /usr/lib/ruby/vendor_ruby /usr/lib/ruby/vendor_ruby_bad










------------------------------


gem update
ERROR:  While executing gem ... (Gem::Exception)
    OpenSSL is not available. Install OpenSSL and rebuild Ruby (preferred) or use non-HTTPS sources

->

$ gem uninstall openssl
Gem openssl-3.0.0 cannot be uninstalled because it is a default gem
Successfully uninstalled openssl-3.0.0
There was both a regular copy and a default copy of openssl-3.0.0. The regular copy was successfully uninstalled, but the default copy was left around because default gems can't be removed.

OK then lol