<template>
    <div>
        <p style="color:#7f8c8d;">心流常在，累计：{{articles.length}} 篇</p>
        <div v-for="article in articles">
            <h2 class="title">
                <router-link :to="article.path">
                    <span class="name">{{ article.frontmatter.title }}</span>
                    <span class="date">{{ article.frontmatter.date?.slice(0,10) }}</span>
                </router-link>
            </h2>
        </div>
    </div>
  </template>
  
  <script>
  export default {
    computed: {
        articles() {
            return this.$site.pages
                .filter(x => x.path.startsWith('/articles/'))
                .sort((a, b) => new Date(b.frontmatter.date) - new Date(a.frontmatter.date));
        }
    }
  }
  </script>

<style>
.title {
    font-size: 1rem;
}
.title a {
    display: block;
    width: 100%;
    overflow: hidden;
}
.name {
    float: left;
}
.date {
    color: #7f8c8d;
    float: right;
}
</style>