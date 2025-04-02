---
layout: page
title: About
---

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Zhicheng Zhong</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            line-height: 1.6;
        }
        .logo-grid {
            display: flex;
            flex-wrap: wrap;
            gap: 10px;
        }
        .logo-grid img {
            height: 40px;
            margin: 5px;
        }
        .profile {
            margin-bottom: 20px;
        }
        .stats {
            margin-top: 20px;
        }
    </style>
</head>
<body>
    <div style="width: 70%; float: left;">
    <div class="profile">

        <h1>I am Zhicheng Zhong👋</h1>
        <p>
            <a href="https://www.linkedin.com/in/zhicheng-z-35805722b/" target="_blank">
                <img src="https://img.shields.io/badge/-Zhicheng-blue?style=flat-square&logo=Linkedin&logoColor=white">
            </a>
            <a href="mailto:zzcjob397@gmail.com">
                <img src="https://img.shields.io/badge/-zzcjob397@gmail.com-c14438?style=flat-square&logo=Gmail&logoColor=white">
            </a>
        </p>
        <ul>
            <li>🔭 Personal Blog: <a href="https://chriszzhong.github.io/">Blog</a></li>
            <li>💬 Ask me about anything, I am happy to help 😊</li>
            <li>📬 How to reach me: <a href="https://www.linkedin.com/in/zhicheng-z-35805722b/">Let's get in touch!</a></li>
        </ul>
    </div>
    <img src="https://raw.githubusercontent.com/rodrigograca31/rodrigograca31/master/matrix.svg" alt="Matrix SVG">
    <h2>Languages and Tools:</h2>
    <div class="logo-grid" style="display: grid; grid-template-columns: repeat(4, auto); gap: 20px; justify-content: start;">
        <img src="https://www.vectorlogo.zone/logos/java/java-horizontal.svg" alt="Java">
        <img src="https://www.vectorlogo.zone/logos/python/python-horizontal.svg" alt="Python">
        <img src="https://www.vectorlogo.zone/logos/w3_html5/w3_html5-ar21.svg" alt="HTML5">
        <img src="https://www.vectorlogo.zone/logos/reactjs/reactjs-ar21.svg" alt="React">
        <img src="https://www.vectorlogo.zone/logos/mysql/mysql-horizontal.svg" alt="MySQL">
        <img src="https://www.vectorlogo.zone/logos/mongodb/mongodb-ar21.svg" alt="MongoDB">
        <img src="https://www.vectorlogo.zone/logos/springio/springio-ar21.svg" alt="Spring">
        <img src="https://www.vectorlogo.zone/logos/angular/angular-ar21.svg" alt="Angular">
        <img src="https://www.vectorlogo.zone/logos/rabbitmq/rabbitmq-ar21.svg" alt="RabbitMQ">
        <img src="https://www.vectorlogo.zone/logos/jenkins/jenkins-ar21.svg" alt="Jenkins">
        <img src="https://www.vectorlogo.zone/logos/git-scm/git-scm-ar21.svg" alt="Git">
        <img src="https://www.vectorlogo.zone/logos/amazon_aws/amazon_aws-ar21.svg" alt="AWS">
        <img src="https://www.vectorlogo.zone/logos/google_cloud/google_cloud-ar21.svg" alt="Google Cloud">
        <img src="https://www.vectorlogo.zone/logos/jupyter/jupyter-ar21.svg" alt="Jupyter">
    </div>

    <div class="stats">
        <a href="https://gitstats.me/ChrisZZhong" target="_blank">
            <img src="https://github-readme-stats.vercel.app/api?username=ChrisZZhong&&show_icons=true&hi&theme=dark&count_private=true&include_all_commits=true" alt="GitHub Stats">
        </a>
    </div>
    <p><strong>Show some ❤️ by starring some of the repositories!</strong></p>
    </div>

    <div style="width: 25%;float: left;position: fixed;left: 75%;top: 40px;bottom: 20px;overflow-x: hidden;overflow-y: scroll;">
        <hearder >
          {% if site.enableToc %}
            <h2 class="post-title">Contents</h2>
            {% include toc.html html=content %}
          {% endif %}
        </hearder>
      </div>

</body>
</html>

{% include comments.html %}
