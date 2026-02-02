# How I built this site, using Go, and Templ

Over the last few months, I've been really loving both the simplicity, and the power that Go provides.  I wanted to put this into a nice app, and rewrite the portfolio prototype I had initially written in React, being served from a go backend.  The problem I ran into was that, for the features I wanted for just a simple portfolio site to showcase the things I'm building or write down a few thoughts in a blog requires a lot of unnecessary complexity and overhead. After some digging I came across this article: [https://mortenvistisen.com/posts/how-to-create-a-blog-using-golang](https://mortenvistisen.com/posts/how-to-create-a-blog-using-golang), it gave me a starting place to build off of for the features that I wanted.

This article can help anyone that would like to do the same.
## The Frontend: Templ

Templ is a **type-safe HTML generation tool** that compiles .templ files into go functions.  This leverages go's type system to generate html at compile time and include them in the single go binary, this makes it fast and gives me all the features that SSR frameworks like Next give you.  

Here's a simple example of a blog article in a .templ file:
```go
type ArticlePageData struct {
    Title 					string
    Content 				string
    FeatureImageURL	string
}


templ ArticlePage(data ArticlePageData) {
    @Base("cjodo | Blog") {
        @components.Container() {
            <div class="article-header"> 
            <h1>{ data.Title }</h1>
            if data.FeatureImageURL != "" {
            <img src={data.FeatureImageURL} alt="">
            }
            </div>
            <article class="line-numbers">
            @templ.Raw(data.Content)
            </article>
        }
    }
}
```

I'm by no means an expert, but it get's the job done.

## The Blog

I prefer writing up notes in just a plain `.md` file, so I looked to just use a markdown parser to parse and serve an html file from that.

I chose to use the `//go:embed` directive in my project to also include the markdown in the binary, allowing for speed and performance, and not having to read from the disk.  
```go
//go:embed files/*.md
var assets embed.FS

```
- The drawbacks of this is that I have to re-compile every time I want to post a new blog.  But for the frequency I'm going to post, I don't think that'll be a problem in the long run.

Then, I just pass in the markdown into a markdown parser, and voila! valid html:

```go
func NewManager() Manager {
    md := goldmark.New(
        goldmark.WithExtensions(
            extension.Table,
            extension.TaskList,
            extension.Strikethrough,
            img64.Img64,
            ),
        goldmark.WithParserOptions(
            parser.WithAutoHeadingID(),
            parser.WithAttribute(),
            ),
        goldmark.WithRendererOptions(
            html.WithHardWraps(),
            html.WithXHTML(),
            html.WithUnsafe(),
            ),
        )

    return Manager{
        posts: assets,
        markdownParser: md,
    }
}

func (m *Manager) Parse(name string) (string, error) {
    source, err := m.posts.ReadFile(postsDir + name)
    if err != nil {
        return "", err
    }

    var htmlOutput bytes.Buffer
    if err := m.markdownParser.Convert(source, &htmlOutput); err != nil {
        return "", err
    }

    return htmlOutput.String(), nil
}

```

For the codeblock formatting, I just use prism.js sent to the frontend to style things and add the line numbers you're seeing.  

## Backend: Go + Echo Server

Nothing fancy here, this is my main.go entrypoint to handle the seperate routes:
```go
func main() {
    e := echo.New();

    e.Pre(middleware.RemoveTrailingSlash())

    db := store.NewDB()

    defer db.Close()

    e.GET("/", func(c echo.Context) error {
        return HomePage(c)
    })

    storage := store.NewArticleStore(db)
    postsManager := posts.NewManager()

    e.Static("/static", "assets")

    blog := e.Group("/blog")

    blog.GET("", func(c echo.Context) error {
        return PostsHome(c, storage, postsManager)
    })

    blog.GET("/:slug", func(c echo.Context) error {
        return PostHandler(c, storage, postsManager)
    })

    e.HTTPErrorHandler = func(err error, c echo.Context) {
        Handle404(err, c)
    }

    e.Logger.Fatal(e.Start(":8080"))
}

```

## DB: Postgres

I really don't need anything fancy for storage, just to store the slug of the posts, and where to find the feature image etc.  Any db would have done just fine.

```sql
create table posts (
    id uuid primary key DEFAULT uuid_generate_v4(),
    created_ts timestamp default CURRENT_TIMESTAMP not null ,
    updated_ts timestamp default CURRENT_TIMESTAMP not null ,
    title varchar(255) not null,
    filename varchar(255) UNIQUE not null,
    slug varchar(255) UNIQUE not null,
    description TEXT not null,
    feature_image text null
);

```
