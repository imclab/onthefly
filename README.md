Browserspeak
============

[![Build Status](https://travis-ci.org/xyproto/browserspeak.svg?branch=master)](https://travis-ci.org/xyproto/browserspeak)
[![Build Status](https://drone.io/github.com/xyproto/browserspeak/status.png)](https://drone.io/github.com/xyproto/browserspeak/latest)

* Package for generating SVG (TinySVG) on the fly
* Can also be used for generating HTML and CSS, or templates for HTML and CSS
* The recommended use is mainly for generating SVG files
* Could be used to set up a diskless webserver that generates all the content
  (which could also be done with templates)

Online API Documentation
------------------------

[godoc.org](http://godoc.org/github.com/xyproto/browserspeak)

Generate content on the fly
---------------------------

<img src="https://raw.github.com/xyproto/browserspeak/master/img/inbrowser.png">

Example
-------

```go
package main

import (
	"fmt"

	"github.com/hoisie/web"
	"github.com/xyproto/browserspeak"
	"github.com/xyproto/webhandle"
)

// Generate a new browserspeak Page (HTML5 and CSS)
func indexPage(cssurl string) *browserspeak.Page {
	page := browserspeak.NewHTML5Page("Demonstration")

	// Link the page to the css file generated from this page
	page.LinkToCSS(cssurl)

	// Add some text
	page.AddContent(fmt.Sprintf("browserspeak %.1f", browserspeak.Version))

	// Change the margin (em is default)
	page.SetMargin(7)

	// Change the font family
	page.SetFontFamily("serif") // sans serif

	// Change the color scheme
	page.SetColor("#f02020", "#101010")

	// Include the generated SVG image on the page
	body, err := page.GetTag("body")
	if err == nil {
		// CSS attributes for the body tag
		body.AddStyle("font-size", "2em")

		// Paragraph
		p := body.AddNewTag("p")

		// CSS style
		p.AddStyle("margin-top", "2em")

		// Image tag
		img := p.AddNewTag("img")

		// HTML attributes
		img.AddAttrib("src", "/circles.svg")
		img.AddAttrib("alt", "Three circles")

		// CSS style
		img.AddStyle("width", "60%")
	}

	return page
}

// Generate a new SVG Page
func svgPage() *browserspeak.Page {
	page, svg := browserspeak.NewTinySVG(0, 0, 128, 64)
	desc := svg.AddNewTag("desc")
	desc.AddContent("Hello SVG")
	svg.Circle(30, 10, 5, "red")
	svg.Circle(110, 30, 2, "green")
	svg.Circle(80, 40, 7, "blue")
	return page
}

// Generator for a handle that returns the generated SVG content.
// Also sets the content type.
func svgHandlerGenerator() func(ctx *web.Context) string {
	page := svgPage()
	return func(ctx *web.Context) string {
		ctx.ContentType("image/svg+xml")
		return page.String()
	}
}

// Set up the paths and handlers then start serving.
func main() {
	fmt.Println("browserspeak ", browserspeak.Version)

	// Connect the url for the HTML and CSS with the HTML and CSS generated from indexPage
	webhandle.PublishPage("/", "/style.css", indexPage)

	// Connect /circles.svg with the generated handle
	web.Get("/circles.svg", svgHandlerGenerator())

	// Run the web server at port 8080
	web.Run("0.0.0.0:8080")
}
```

Version, license and author
---------------------------

* Version: 0.6
* License: MIT
* Alexander Rødseth

