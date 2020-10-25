---
- Follow `sentry & kibana` to aware the issue as soon as possible
- If the issue related to DB or can not `reproduce` on the localhost, we must add code for debug or asking team-leader or someone who can SSH to server and debug via `django-shell`
  + Grab input parameters from `sentry/kibana/cloudwatch` if possible.
  + You must know which `command/function` need to be check first.
  + Then give that `command/function` for team-leader to help debug on server if possible

- Use `Kibana APM & AWS slowquery` to detect the performance problem.

---
- Request API with an additional `HEADER` called `PROFILLING=1`, It will start profile your request then put the profiler report to our S3.
    + You need to request someone who can access the report folder on S3 to get `.prof` file.
    + Use some 3rd tool to visualize your report file to know the code flows and detect the issue.
    + Some tools I'm using for this purpose:
      + https://jiffyclub.github.io/snakeviz/
      + https://github.com/jrfonseca/gprof2dot
    + It would be helpful for case you want to improve your code to make it faster and smarter :D.

---
- **Other Tips?**

![Another solution](https://lawfulrebel.com/wp-content/uploads/2017/09/Thinking-Image-10.jpg)
