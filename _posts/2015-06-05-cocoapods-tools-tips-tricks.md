---
layout: post
title: CocoaPods tools, tips & tricks
permalink: cocoapods-tools-tips-tricks
comments: true
---
In the spirit of [Ash Furrow's Teaching & Learning](http://ashfurrow.com/blog/teaching-learning/), here's a compilation of CocoaPods pointers I've picked up after maintaining private pods daily.
 
## Using private spec repo
It's something to consider, even if you don't maintain private pods, in the following situations:

- using _bleeding edge_ version of the library, that isn't deemed stable yet (not tagged, no podspec for this version yet),
- there's a bug fix that you just need, but fix isn't in version referenced in a podspec and maintainers don't want to update patch version,
- having fork with functionality you specifically need/maintainer not appreciating the pull request.
 
And the usual use cases for this:

- maintaining private library (you can get by using `:git`, however...),
- private library having dependency on another private library.
 
<!--more-->
 
When pod version is referenced by `:git` or `:head:`, each time `pod update` runs it pre-downloads dependencies. Doing:
 
`pod 'PEPhotoCropEditor', :git => 'https://github.com/mr-v/PEPhotoCropEditor'`
 
results in running:
 
`git clone https://github.com/mr-v/PEPhotoCropEditor --single-branch --depth 1`
 
This command, depending on your network connection, can take from ~10 seconds to a minute on a bad Wi-Fi. Repo is stable and doesn't really change, so why should we pull all this stuff each time? Using private specs repo solves this.  There's a nice [official guide](https://guides.cocoapods.org/making/private-cocoapods.html) on how to setup and use such repo. After you're done with setup, add new spec repo as `source` to a `Podfile`, [e.g.](https://github.com/artsy/eidolon/blob/master/Podfile#L2):
 
`source 'https://github.com/artsy/Specs.git'`
 
To use custom version of the library:

- configure podspec to point to specific commit, here commit/tag/branch can be used without worrying about triggering unnecessary downloads,
- push podspec to private repo (`pod repo push reponame`).
 
Now private dependency can be specified like public one, without referencing git repository directly:
 
`pod 'PFXAmazingLibraryDoingStuff', '~> 0.0.1'`
 
`pod update` updates only what is really needed.

### Linting
When linting the private spec with dependency on another private lib, be sure to [specify private spec repo](http://stackoverflow.com/a/27305019) in `--sources` switch.
 
`pod spec lint --sources='https://github.com/suchprivatespecrepo/Specs.git,https://github.com/CocoaPods/Specs'`
 
### --allow-warnings
When pushing a pod with `pod repo push reponame`, `--allow-warnings` can be a lifesaver when you just don't want to deal with writing summary for internal library or podspec isn't pointing to a tag.

## Updating podspec versions
Instead of updating podspec manually use a npm package called [podspec-bump](https://www.npmjs.com/package/podspec-bump). To see how podspec file would look after updating patch version, just run the command without any parameters. My preffered way of working is to run `podspec-bump -w` (bumps patch version, writes change to file) and then check changes with quick `git diff`. `-i` switch for bumping major and minor versions is available. There's also `--dump-version`, which can be put to work like [this](http://efcl.info/2014/08/22/cocoapods-podspec-release-tools/):

```bash
podspec-bump -w # update version
git commit -am "bump `podspec-bump --dump-version`" 
git tag "`podspec-bump --dump-version`"
git push --tags
pod trunk push 
```
 
## Hey we don't support those languages, do we?
If you are using libraries that have [lots of localizations](https://github.com/mattt/FormatterKit/tree/master/Localizations) and support languages you don't, you may encounter a problem with App Store displaying that app supports those languages as well. To mitigate it use CocoaPods [cocoapods-prune-localizations plugin](https://github.com/dtorres/cocoapods-prune-localizations), which allows to specify supported language codes in a `Podfile`:
 
`plugin 'cocoapods-prune-localizations', {:localizations => ["en"]}`
 
On each install/update those unwanted localizations will be pruned. Aside from fixing info on the App Store, it has benefit of removing unused resources from your app.
 
## Defining macro in a third party dependency
Macros can be great, macros can be bad. In the end it boils down to how you use them. In my case I needed to add support for `ARAnalytics` to `AFNetworkActivityLogger`. Figured out a way to do it in a quick and dirty fashion (switching `NSLog` calls in `AFNetworkActivityLogger` to `ARLog`). Following [Stack Overflow advice](http://stackoverflow.com/a/27138078) it can be achieved by adding `post_install` hook to a `Podfile`:
 
```ruby
post_install do |installer_representation|
  installer_representation.project.targets.each do |target|
      target.build_configurations.each do |config|
      if target.name == 'Pods-AFNetworkActivityLogger'
      puts "Setting preprocessor macro for #{target.name}"
        puts "#{config} configuration..."
        puts "before: #{config.build_settings['GCC_PREPROCESSOR_DEFINITIONS'].inspect}"
        config.build_settings['GCC_PREPROCESSOR_DEFINITIONS'] ||= ['$(inherited)']
        config.build_settings['GCC_PREPROCESSOR_DEFINITIONS'] << 'NSLog=ARLog'
        puts "after: #{config.build_settings['GCC_PREPROCESSOR_DEFINITIONS'].inspect}"
        puts '---'
	   end
      end
    end
end
```