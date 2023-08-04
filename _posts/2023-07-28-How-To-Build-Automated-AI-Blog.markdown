---
layout: post
title:  "How to Build a Static Blog With Automated A.I. Generated Content"
date:   2023-07-30
categories: A.I Automation Python
---

![SkinnyDip Blog Index Page](/assets/blog_index.png)

[Demo Site](https://www.skinnydipblog.com/)

By leveraging OpenAI’s API I was able to create a static blog with self-generating content including images and text. Since the site is static, hosting comes at no cost. Moreover, with OpenAI offering free credits to new developers, you too can create your own blog at minimal expense. Let me walk you through the steps I took to accomplish this.

# 1. Choose a Static Site Generator. 

A static site generator (SSG) is a software tool that automates the process of creating static websites. It takes input in the form of templates, content, and configuration, and then generates a collection of static HTML, CSS, and JavaScript files that constitute the final website. The generator processes the input data and combines it with pre-defined templates, applying the site’s layout, design, and structure consistently across all pages. This eliminates the need for a traditional server, which happens to be the most costly aspect of site hosting. Services like Netlify or Surge offer free hosting for static sites, making it a cost-effective and efficient solution for website deployment.

Any static site generator could be used for this project, some popular options include [Hugo](https://gohugo.io/), [Jekyll](https://jekyllrb.com/), and [Eleventy](https://www.11ty.dev/). I went with [Gatsby.js](https://www.gatsbyjs.com/). Gatsby combines React.js and GraphQL to create blazing-fast and feature-rich websites. Gatsby also comes with an extensive [starter library](https://www.gatsbyjs.com/starters/), saving us the trouble of starting from scratch. I used the [blog starter template](https://www.gatsbyjs.com/starters/gatsbyjs/gatsby-starter-blog/). You can fork a copy of the code and automatically deploy it to Gatsby Cloud (Gatsby’s CDN provider) using this [link](https://www.gatsbyjs.com/dashboard/deploynow?url=https%3A%2F%2Fgithub.com%2Fgatsbyjs%2Fgatsby-starter-blog).

After cloning the repository, installing the dependencies and running `gatsby develop` in your terminal, you should see this page in your browser.

![Gatsby Index Page](/assets/gatsby.png)

I wrote some custom CSS in my project, but this is not a CSS tutorial, so check out my code if you want to copy the styles. I also tweaked the default GraphQL query to set a featured image for each blog post. You can read more about this feature in [Gatsby’s documentation](https://www.gatsbyjs.com/docs/how-to/images-and-media/working-with-images-in-markdown/#featured-images-with-frontmatter-metadata). Regardless, everything that follows should work with the vanilla starter without making any of these changes to the base code.

# 2. Install Dependencies and Connect with OpenAI API.
Using a package manager, install OpenAI’s library in your preferred language. My project used Python.

`pip install openai`

OpenAI requires an API key. [Sign up is easy](https://auth0.openai.com/u/signup/identifier?state=hKFo2SByelVwaEVoN096elQxRUZQNVNUWHRCNU4wZlhMRHotWaFur3VuaXZlcnNhbC1sb2dpbqN0aWTZIDBNZWx1S2Z0WDRBZXh0aTY2Y0VxT1pvZjZPdC1jTmJYo2NpZNkgRFJpdnNubTJNdTQyVDNLT3BxZHR3QjNOWXZpSFl6d0Q), and new developers are granted $5 credit to experiment with the API. This should be plenty for building this project. (I only used up about $1 of my free credit.)

This API key should not be used directly in your code as others might be able to access it and drain credits from your account. Instead, save it as an environment variable using this [guide](https://chlee.co/how-to-setup-environment-variables-for-windows-mac-and-linux/).

We will also need to install Python’s requests library to interact with our image output.

`pip install requests`

# 3. Configure Prompts and Generate Content.
Now for the fun part. It’s time to configure your own prompts for producing the blog’s content. For my project, I decided to create a blog of nature poetry inspired by the work of Robinson Jeffers.

I started with a glue function responsible for coordinating the rest of our work:

```
def create_content():
    path = "../skinnydip/content/blog/"
    date = datetime.datetime.now().strftime("%Y-%m-%d")    
    new_post_path = os.path.join(path, date)
    os.mkdir(new_post_path)
    get_img(new_post_path)
    title, poem = get_poem()
    create_post(title, poem, date, new_post_path)
```

First, I hardcoded the path to the blog’s content files, and created a directory using the date as a label to keep things organized. This directory will contain all images and posts that we generate.

We need to generate three pieces of data: The poem itself, a title for the poem, and a featured image. Let’s start with the poem and title.

```
def get_poem():
    poem_completion = openai.ChatCompletion.create(model="gpt-3.5-turbo", messages=[{"role": "user", "content": "Write a short nature poem modeled after Robinson Jeffers."}])
    poem = poem_completion.choices[0].message.content
    title_completion = openai.ChatCompletion.create(model="gpt-3.5-turbo", messages=[{"role": "user", "content": "Provide a title for this poem. {poem}"}])
    title = title_completion.choices[0].message.content
    return (title, poem)
```

To generate the poem and title I decided to use the chat completion function to take advantage of the newer (and cheaper) models available in the OpenAI ecosystem. Required inputs include the model that we want to work with (in my case, GPT 3.5), a messages list containing the role of the message creator, and the content of the message. Feel free to experiment with the roles available (“system”, ”user”, “assistant”, or “function”). I received good results with the role set to “user”. For content I provided a basic prompt: “Write a short nature poem modeled after Robinson Jeffers.” Then I parsed the returned object for the content I needed and fed this into a second request to get a title for the poem.

The process of obtaining an image was similar. As part of my aim to maximize automation, I delegated the responsibility of creating an image prompt to the chat completion function which was used in the previous code snippet.

```
def get_img(path):
    get_img_prompt = f"In 20 words or less, suggest a prompt for creating an inspiring nature image. Suggest visual artists by name."
    img_prompt_completion = openai.ChatCompletion.create(model="gpt-3.5-turbo", messages=[{"role": "user", "content": get_img_prompt}])
    img = openai.Image.create(
      prompt= img_prompt_completion.choices[0].message.content,
      n=2,
      size="1024x1024"
    )
    url = img.data[0].url
    r = requests.get(url, allow_redirects=True)
    open(f'{path}/header.png', 'wb').write(r.content)
```

I found the prompts provided in my initial attempts to be too long. So I added the additional restriction of limiting outputs to 20 words or less. The quality of the image is greatly improved by naming artists whose style we would like to emulate, so I also included this as a condition in my prompt. Then I fed this prompt into the image generation function provided by the API. This function takes in our prompt, the number of images to generate (n), and the size of the image that we’re creating, and outputs a URL where the image is stored. I then used Python’s request library to download the image and write it to file.

# 4. Automate using Python.
With the content for our post generated, now we need to create the files that Gatsby will parse to create our blogpost. Our starter parses markdown files for the content of our blog. We will use Python’s built in OS module to create these files.

`import os`

The most critical data used to parse our post and create the blogpost’s page is contained in the header of our markdown file. It should look something like this:

```
---
title: Whispers of the Night
date: "2023-07-27"
featuredImage: header.png 
---
```

In my code, I used f-strings to insert the title and date using the data passed down from the create_content function. Then I appended the markdown for displaying our image, followed by the poem.

```
def create_post(title, poem, date, path):
    f = open(f'{path}/index.md', "w")
    header = f'---\ntitle: {title}\ndate: \"{date}\"\nfeaturedImage: header.png \n---'
    f.write(header)
    f.write('\n\n')
    f.write('![nature image](./header.png)')
    f.write('\n\n')
    f.write(poem)
    f.close()
```

# 5. Deploy from Github to CDN Provider
The aspect I love most about working with static sites is the effortless deployment process. [Gatsby Cloud](https://www.gatsbyjs.com/products/cloud/) stands out as the recommended CDN for Gatsby sites. After [sign up](https://www.gatsbyjs.com/dashboard/signup), all we have to do is link our Github repository, and Gatsby Cloud takes care of the rest. By default, the domain name will be [site-name].gatsbyjs.io. If you prefer to use a custom domain, you can easily achieve this by heading to the hosting tab in the site settings section and following the [provided instructions](https://www.gatsbyjs.com/docs/how-to/cloud/adding-a-custom-domain/) to modify the CNAME records with your preferred domain provider.

# Conclusion
And that’s it! You have a fully automated A.I. blog. For any questions, comments, or if you catch any errors in my code, reach out on [Twitter](https://twitter.com/justgnnr). Below I’ve linked to my code and some resources that I found useful in putting the project together. Thanks for reading!

# Resources
[Demo Site](https://www.skinnydipblog.com/)

[Github Repo for Blog](https://github.com/justgnnr/skinnydipblog)

[Github Repo for Content Generator](https://github.com/justgnnr/blog_generator)

[Gatsby Documentation](https://www.gatsbyjs.com/docs/)

[OpenAI Documentation](https://platform.openai.com/docs/introduction)