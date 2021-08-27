# A new frontend for the Environment and Health Data portal

This repository contains a new prototype of the Environment and Health Data Portal. You can view a staged development version [here](https://nycehs.github.io/ehs-data-portal-frontend-temp/).

## How you can help

In the spirit of free software, everyone is encouraged to help improve this project.  Here are some ways you can contribute.

- Comment on or clarify [issues](https://github.com/nycehs/ehs-data-portal-frontend-temp/issues)
- Suggest new features
- Write or edit documentation
- Write code (no patch is too small)
  - Fix typos
  - Add comments
  - Clean up code
  - Add new features

## Requirements

You will need the following things properly installed on your computer.

- [Git](https://git-scm.com/)
- [Hugo](https://gohugo.io/) 

## Development

- On your local, the server with `hugo serve --environment local`
- Develop on branches labelled hotfix-, content-, or feature-. 
- When merging branches into main, run a new build on main to /docs.

## Architecture

Development is largely done in the /content and the /themes directories. Functionality worth noting for ongoing development:

### Navigation
The nav menu will highlight the content area (e.g., "Data Stories") when the user is on that area's landing page, or on a subpage within that directory. For this to work, each markdown file (especially subpages) needs the following in the front matter:

```
menu:
    main:
        identifier: '02'
```

Use 01 for subpages of the home page, 02  for data stories, 03 for the data explorer, 04 for neighborhood reports, and 05 for Key Topics (per config.toml).

### Data Stories banner images
Data stories include a banner image. This image is added to static/images. It should be called ```ds-[storyname].jpg```. The image should be referenced in the front matter this way:
```
image: ../../images/ds-assaults.jpg
```

It is referenced via in-line CSS in themes/dohmh/layouts/data_stories/single.html.

### Indicators
Indicators can be displayed on subtopic pages. Indicators are stored as json within subtopic content markdown file's front matter - see [asthma.md](https://github.com/nycehs/ehs-neighborhoodprofiles/blob/main/content/data_explorer/asthma.md) or below:

```
indicators: {
    {
        "name" : "ED visits (adults)",
        "URL": "http://a816-dohbesp.nyc.gov/IndicatorPublic/VisualizationData.aspx?id=2380,4466a0,11,Summarize"
    },

    {
        "name" : "ED visits (age 0-4)",
        "URL": "http://a816-dohbesp.nyc.gov/IndicatorPublic/VisualizationData.aspx?id=2048,4466a0,11,Summarize"
    }
}
```


Then, the template page loops through the Indicator JSON and displays each indicator:

```
{{ range .Params.indicators}}
    <a href="{{.url}}" onclick="protoPopup()">{{.name}}</a>
    <hr>
{{end}}
```

### Ranging through items in another content section
This code works on any template page to range through items in another content section. For example, placed on key_topics/section.html, it ranges through all of the Site's Pages that are in the data_stories section, and prints the Title.

```
    {{ range where .Site.RegularPages "Section" "data_stories" }}
        {{ .Title }}<br>
    {{end}}
```

This is more complex code that looks for the intersection of two areas' categories field. As a reminder, we use categories to tag matter with their **key topics**. 

```
    <!--Establishes two variables-->
    {{ $page_link := .Permalink }}
    {{ $cats := .Params.categories }}
    <!--Ranges through the section we want to ingest into this page-->
    {{ range where .Site.RegularPages "Section" "data_explorer" }}
    <!--Places the contents of that range, ., into a variable called $page-->
    {{ $page := . }}
    <!--Defines a variable as the intersection of the ranged pages .Params.categories, and this page's-->
    {{ $has_common_cats := intersect $cats .Params.categories | len | lt 0 }}
    <!--Excludes this page-->
    {{ if and $has_common_cats (ne $page_link $page.Permalink) }}
    <li><a href="{{ .URL}}">{{ .Title }}</a></li>
    {{ end }} 
    {{ end }}
```

Data stories, Key Topics, and Data Explorer (subtopics) markdown should have the following in the frontmatter:
- categories: ["key topic 1", "key topic 2"]
- keywords: a field used to populate information for the site search function
- tags: a freeform category we are not yet using

### Using custom layouts
If a file contains ```layout: custom``` in the frontmatter,  Hugo will look for a layout named ```custom.html``` in the same directory structure within ```/themes/dohmh/layouts```. Use this for one-off data features like the Air Quality Explorer. For Key Topic landing pages, use ```layout: single``` to force these pages to display based on the ```single.html``` template instead of the list template (```section.html```).

### Data visualization shortcodes
Shortcodes for Datawraper and Vega/Vega-Lite both exist. With shortcodes, you enter simple code in markdown that inserts components into pre-written code. 

To embed a Vega/Vega-Lite visualization, simply add this:. The shortcut inserts the id and the spec into standard V/VL code. Store chart specifications in ```static/visualizations/spec```. Additionally, the markdown file needs ```vega: true``` which adds Vega libraries to ```head.html```. 
```{{< vega id="uniqueDivID" spec="../../visualizations/spec/bartest.vl.json" >}}```

For Datawrapper, the shortcode is:
```{{< datawrapper "Title" "chartID/version/" "Height" >}}```

## Testing and checks

- **Update this list** - with the project's QA/QC rules and any testing information; below are examples only

- **ESLint** - We use ESLint with Airbnb's rules for JavaScript projects
  - Add an ESLint plugin to your text editor to highlight broken rules while you code
  - You can also run `eslint` at the command line with the `--fix` flag to automatically fix some errors.

- **Testing**
  - run `ember test --serve`
  - Before creating a Pull Request, make sure your branch is updated with the latest `develop` and passes all tests

## Deployment

{Description of what type of hosting environment is required, and steps for how you deploy -- e.g `git push dokku master`.}

## Contact us

You can comment on issues and we'll follow up as soon as we can. 


## Communications disclaimer

With regard to GitHub platform communications, staff from the New York City Department of Health & Mental Hygiene are authorized to answer specific questions of a technical nature with regard to this repository. Staff may not disclose private or sensitive data. 
