---
layout: course_layout
pagination:
    data: coursesapifull.courses
    size: 1
    alias: course
permalink: "courses/{{ course.name | slug | removeInvalidChars}}/"
page:
    type: course
---
{{ courseapi | json }}
