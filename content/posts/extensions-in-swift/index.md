---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Extensions in Swift"
subtitle: ""
summary: ""
authors: [Dylan Powers]
tags: []
categories: []
date: 2019-08-13T13:25:02-06:00
lastmod: 2019-08-13T13:25:02-06:00
featured: false
draft: false

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: ""
  focal_point: ""
  preview_only: true

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: []
---

Today we’re going to talk about what I think is Swift’s coolest feature — extensions.

Extensions allow you to add functionality to an already existing class. In one of my recent projects, I implemented an [OCR](https://en.wikipedia.org/wiki/Optical_character_recognition) model that could read text (specifically, a credit card number) from a picture that the user took with the back camera. Unfortunately, if the image wasn’t oriented upwards, the OCR fails, as we would expect. As such, I needed a convenient way to change the orientation of the image with ease.

![Credit cards](featured.jpg)

I quickly found some [code](https://gist.github.com/schickling/b5d86cb070130f80bb40) that could easily convert an image (thanks to GitHub user Johannes Schickling for this). Here are the workflows with and without extensions:

## Without Extensions

Without extensions, the workflow is similar to that of any typical Object-Oriented language. The developer would create a class that performed the orientation fix, and name it accordingly:
	
	class ImageOrientationFixer {
	    init() { }
	}

The user would then write a method that takes the image as a parameter, and returns an image with the new fixed orientation.
	
	class ImageOrientationFixer {
	    init() { }

	    func changeOrientationToUpright(_ image : UIImage) -> UIImage {
	        // code to correct the orientation
	    }
	}

There are two major problems with this:

1. In every subsequent class that requires this functionality, we would have to instantiate a new `ImageOrientationFixer` . A quick solution would be to just make the method `static` , which leads into

1. This requires passing the image object to the fixer on every call. Any situation where a client calls a function, passes it a mutable object, and receives the same (modified) object in return is a **blatant** code smell, and indicative of a poorly designed API.

Alternatively, the method *could* return void and simply change some attributes of the image along the way. However, since the callee can still change the image at will, this is still a code smell.

## With Extensions

Extensions fix both of the problems outlined above. Instead of creating a new class that does this, we can **add that functionality to Swift’s `UIImage` class.** This means that we can just directly call our new function on the image object itself. This solves the problems outlined above, because the client then never gives up control of the image object. The declaration looks like this:

	extension UIImage {
	    func changeOrientationToUpright() {
	        // code to correct the orientation
	    }
	}

Then, anytime I want to change the orientation of some `image` object that I have, I simply call `image.changeOrientationToUpright()` and all my dreams come true.

Extensions have a multitude of use cases, all of which make any mobile developer’s life much easier. Most importantly, they eliminate cases where a client would otherwise pass a mutable object to another class for processing. If you haven’t already, give Extensions a shot!

