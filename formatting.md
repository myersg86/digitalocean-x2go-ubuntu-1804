## Formatting

DigitalOcean tutorials are formatted in the Markdown markup language. [Daring Fireball](http://daringfireball.net/projects/markdown/syntax) publishes a comprehensive Markdown guide if you're unfamiliar with it. DigitalOcean also uses some [custom Markdown](https://www.digitalocean.com/community/markdown). Examples of our custom Markdown are in the appropriate sections below.

### Headers

Each section of our tutorials has a corresponding header: the title should be an H1 header; the introduction should be an H3 header; the goals, prerequisites, steps, and conclusion should have H2 headers. You can see this format in our [Markdown article templates](https://github.com/do-community/do-article-templates).

For procedural tutorials, step headers should include step numbers (numerical) followed by an em dash (**—**). Step headers should also use the gerund, which are **-ing** words. An example step header is **Step 1 — Installing Nginx**.

Use H3 headers sparingly, and avoid H4 headers. If you need to use subheaders, make sure there are two or more headers of that level within that section of the tutorial. Alternatively, consider making multiple steps instead.

### Line-level Formatting

**Bold text** should be used for:

- Visible GUI text
- Hostnames and usernames, like **wordpress-1** or **sammy**
- Term lists
- Emphasis when changing context for a command, like switching to a new server or user

*Italics* should only be used when introducing technical terms. For example, the Nginx server will be our *load balancer*.

In-line code formatting should be used for:

- Command names, like `unzip`
- Package names, like `mysql-server`
- Optional commands
- File names and paths, like `~/.ssh/authorized_keys`
- Example URLs, like `http://example.com`
- Ports, like `:3000`
- Key presses, which should be in ALL CAPS and use a plus symbol, **+**, if keys need to be pressed simultaneously, like `ENTER` or `CTRL+C`

### Code Blocks

Code blocks should be used for:

- Commands the reader needs to execute to complete the tutorial
- Files and scripts
- Terminal output
- Interactive dialogues that are in text

Excerpts and omissions in files can be indicated with ellipses (**. . .**). If most of a file can be left with the default settings, it's usually better to show just the section that needs to be changed.

#### Code Block Prefixes

Do not include the command prompt in the code block. Instead, use DigitalOcean's custom Markdown for non-root user commands, root user commands, and custom prefixes, respectively:

```
​```command
sudo apt-get update
​```

​```super_user
adduser sammy
​```

​```custom_prefix(mysql>)
FLUSH PRIVILEGES;
​```

```

This is how the above examples look when rendered:

> ```
> sudo apt-get update
> ```
>
> ```
> adduser sammy
> ```
>
> ```
> FLUSH PRIVILEGES;
> ```

#### Code Block Labels

DigitalOcean's Markdown also includes labels and secondary labels. You can add labels to code blocks by adding a line with `[label Label text]` or `[secondary_label Secondary label text]` anywhere in the block.

Use labels to mark code blocks containing the contents of a file with a filename. Use secondary labels to mark terminal output.

Labels look like this when rendered:

This is the label text

```
This is one line of the file
This is another line of the file
. . .
This is a line further down the file
```

Secondary label example:

```
This is the secondary label textThis is some output from a command
```

#### Code Block Environment Colors

DigitalOcean's Markdown allows you to color the background of a code block by adding a line with `[environment name]` anywhere in the block. The options for `name` are `local`, `second`, `third`, `fourth`, and `fifth`.

This is a local server command example:

```
ssh root@your_server_ip

```

These are non-primary server command examples, useful for multi-server setups:

```
echo "Secondary server"

```

```
echo "Third server"

```

```
echo "Fourth server"

```

```
echo "Fifth server

```

### Notes and Warnings

The DigitalOcean Markdown parser allows for custom **note** and **warning** code blocks to be used to display very important text.

Here's a Markdown example of a note and a warning (this is an image):

![Notes and Warnings](https://assets.digitalocean.com/articles/do_formatting/note_warning.png)

Here's the rendered result:

**Note:** This is a note.

**Warning:** This is a warning.

### Variables

Highlight any items that need to be changed by the reader, like example URLs or modified lines in configuration files. You can do this by surrounding the word or line with our custom **<^>** Markdown. Note that you cannot highlight multiple lines with one pair of symbols, so you need to highlight each line individually.

If you reference a variable in a context where you would normally also use `in-line code` formatting, you should use `both styles`. Make sure your tutorial is as accessible as possible by using language like "highlighted in the preceding code block" instead of "highlighted in red above".

Avoid language like "highlighted in red below"

### Images and Other Assets

Images can quickly illustrate a point or provide additional clarification in a step. Use images for screenshots of GUIs, interactive dialogue, and diagrams of server setups. Don't use images for screenshots of code, configuration files, output, or anything that can be copied and pasted into the article.

If you're including images in your tutorial, please follow these guidelines:

- Include descriptive alt text so readers using a screen reader can rely on the alt text rather than the image.
- Use the `.png` file format
- Host images on [imgur](http://imgur.com/)
- Make the image with as short a height as possible

If you make a mockup of a diagram for your tutorial, we will create a diagram in the DigitalOcean style. We'll also upload all images to DigitalOcean servers at publication time.

Here's a Markdown example for including images in your tutorial:

```
![Alt text for screen readers](http://imgur.com/your_image_url)
```

Occasionally, you will want the reader to have access to a configuration file that is too long to display in the main body of the tutorial. DigitalOcean will host this file on our assets server. You can use standard link formatting to link to the file.
