```java
public void testPullFileContent() throws Exception {
    JSONObject json = GitHubApiTool.pull(
            new String[]{"vintsie", "notebook", "_2014/_2014_01_03_项目奖又被忽悠了.md"});
    String content = json.getString("content");
    String html = new PegDownProcessor().markdownToHtml(content);
    System.out.println(html);
    System.out.println(content);
}
```
