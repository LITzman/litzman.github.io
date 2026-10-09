
# Vibe Coding a Memoir

### Background
Last week, an amazing project appeared in my feed: https://artmuseum.artfrompixels.com
It shows an interactive timeline of artists, periods and styles and once you select an artist and enter his gallery something ridiculously cool happens:
 *You enter a virtual 3D museum of the artist's works.*
Navigating the museum is intuitive and there's even an audio guide and a place to sit.

The fact that it's possible to load something that feels like Triple-A video game in the browser and have it run so smooth made me genuinely happy.

The only downside:
I honestly don't care about none of the artists on display there.

### Modernizing The Pipeline
I want to use the "museum" component of this project to show media from an Instagram page.
This begins by having Claude decouple the museum component, and adapting the underlying data format from *context-rich* Wikipedia metadata to your *context-poor* Instagram slop:
```
{
	"account": {
		"username": "LITzman",
		"fullname": "Yaakov LITzman",
		"url": "https://www.instagram.com/LITzman/"
	},
	"pictures": [
		{
			"image_name": "<ID>.jpg",
			"date": "2015-01-02",
			"description":  "...",
			"width": 640,
			"height": 640,
			"url": "https://www.instagram.com/p/<ID>/"
		},
    ...
    ]
},
```

Another open source project called [gallery-dl](https://github.com/mikf/gallery-dl) makes quick work of scraping all images and metadata from Instagram - it even steals your browser's cookies so you don't have to manually login.

By filling `museum.config.json` with details such as "whose account to scrape", "what's the color of the walls" and "which time period should we show on the walls" you can customize your own little museum, and running `npm run deploy` does **ALL**  the magic - 

 - Scrapes all data (locally on your machine)
 - Creates a minified build of the app
 - Deploys it to a (free) Github-pages hosted site.

### The Rationale
In September 2022 I lost my father.
He was a graphic designer and has done quite a bit of web development.
Besides his professional work, he was an actual artist - designer, painter, woodworker, and a photographer.
His Instagram page is full of some of his best works and serves as a little memoir.

For a long while I wanted to display these works on the web in an interactive format that serves his spirit and style better, and now it's finally online.
## [See Arie's Museum](https://litzman.com/arie-museum/)


All credit and a huge thanks to [Jason Tragakis](https://github.com/iasonastr) for building this awesome project.
