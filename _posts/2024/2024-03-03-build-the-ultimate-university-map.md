---
title: Build the Ultimate University Map
image: /assets/img/default-banner.jpg
author: jack
date: 2024-03-03 20:55:00 +0800
categories: [reactJS]
tags: [reactJS, leaflet]
math: false
pin: false
---

Hey there, fellow developers and map enthusiasts! Welcome to the Dev Blog for [University Map](https://university-map.github.io), where we're crafting the ultimate university map experience. If you're passionate about maps, user interfaces, and making information accessible, you're in the right place. Let's dive into what Project Uni is all about and how you can get involved.

## What is University Map?

University Map is a side project dedicated to creating a comprehensive map of universities around the world. Our goal is to make it easy for students, researchers, and anyone curious about higher education to explore universities, their locations, and key information about them. Whether you're planning your next academic journey or simply exploring the vast world of education, this project has you covered.

## Feature Highlights
- Comprehensive University Profiles
- Search Functionality
- Internationalization

## Tech Stack
- ReactJs
- Mantine
- Leaflet + OpenStreetMap

## The Journey

Firstly, I've only worked with Vue.js in my professional experience. However, during my research, I noticed that most articles related to Leaflet use React.js. I believe using React.js might make problem-solving easier. When I visited the React.js official website, they suggested using frameworks like Next.js or Remix. After watching this easy-to-follow [tutorial](https://www.youtube.com/watch?v=JBTKlcUJXQ4), I successfully initialized a map and marked National Cheng Kung University on it. Here's the corresponding commit: [5cfad27](https://github.com/university-map/university-map-nextjs/tree/5cfad278f3fc1174973dcdee9a2ca37258f9266e).

![image1](https://github.com/university-map/university-map/assets/46962923/a91a19aa-cef6-48b0-82bf-c2b31868c824)

Next, I incorporated Ant Design to create a fixed information panel on the left side. This panel is used to display university banners and information. I chose Ant Design as my UI framework for its robust documentation and easiness. With the university data described in YAML format, I can now dynamically populate the information panel with the university's information:

```yaml
# /public/universities/Taiwan/National Taiwan University/en.yml
name: "National Cheng Kung University"
address: "No.1, University Road, Tainan City 701, Taiwan (R.O.C)"
locations:
  - name: "Main Campus"
    coordinates: [22.9987772261614, 120.21648234565666]
  - name: "Guiren Campus"
    coordinates: [22.93788805281445, 120.27485576268575]
acronyms: ["NCKU"]
banner: "https://web.ncku.edu.tw/var/file/0/1000/img/495849468.jpg"
introduction: >-
  National Cheng Kung University was originally established under Japanese colonial government in January 1931 as the Tainan Technical College.
  It was upgraded to a provincial university in 1956 as Provincial Cheng Kung University, named after Koxinga, a Chinese military leader who drove the Dutch East India Company from Taiwan and founded the Kingdom of Tungning.
  In 1971, the university became a national university and was renamed to National Cheng Kung University.
gallery:
  [
    "https://web.ncku.edu.tw/var/file/0/1000/img/2638/541084137.jpg",
    "https://web.ncku.edu.tw/var/file/0/1000/img/495849468.jpg",
    "https://web.ncku.edu.tw/var/file/0/1000/img/0111_Sel-044.jpg",
  ]
```

In addition, when this webpage first loads, it will first load the Markers of all universities through a `locations.json`:

```json
[
  {
    "name": "National Cheng Kung University",
    "country": "Taiwan",
    "location": [
      {
        "name": "Main Campus",
        "coordinates": [
          22.9987772261614,
          120.21648234565666
        ]
      },
      {
        "name": "Guiren Campus",
        "coordinates": [
          22.93788805281445,
          120.27485576268575
        ]
      }
    ]
  },
  {
    "name": "National Taiwan University",
    "country": "Taiwan",
    "location": [
      {
        "name": "Main Campus",
        "coordinates": [
          25.017491194856326,
          121.53968742646664
        ]
      }
    ]
  }
]
```

Here's the corresponding commit: [23fa274](https://github.com/university-map/university-map-nextjs/tree/23fa27484967616db8a8aa72c42f3a5e32fa209f).

![image2](https://github.com/university-map/university-map/assets/46962923/8e3fd05d-9bcc-426a-b00d-192511e9b782)

After realizing that Ant Design's style didn't quite match the desired effect, I switched the UI to Mantine. Next, I added a vertical navbar on the left side of the screen. Inside the navbar, there's a language picker used to switch between languages with next-intl, and a link to the GitHub repository of this project. The setup for next-intl was done with reference to this easy-to-follow [tutorial](https://www.youtube.com/watch?v=pKolfZFW3gs).

Here's the corresponding commit: [3bfe12a](https://github.com/university-map/university-map-nextjs/tree/3bfe12a2837eea427b7fc1e32b52fe4c094c191b).

![image3](https://github.com/university-map/university-map/assets/46962923/1c9cb4d1-a1b8-4262-910d-a6c33910e97f)

At this point, I believed that the development had reached a milestone, and I was ready to deploy it to GitHub Pages. However, I encountered a critical issue: after the project was built, it wasn't simply a static website. The problem arose from the dynamic routing structure, such as `/:lang/university/:country/:university`, causing the browser to send requests to GitHub Pages' web server each time a route was switched. Additionally, some Next.js functionalities or middleware might require backend processing. As a solution, I decided to rewrite the entire project using Vite + ReactJS. I replaced next-intl with i18next and implemented a hash router. Finally, after these changes, my website could be successfully deployed on GitHub Pages.

Next, I utilized id_ed25519 as the deploy key for the GitHub Pages repository. The GitHub workflow performed the deployment using the following script:

```bash
echo "$BOT_PRIVATE_SSH_KEY" > id_ed25519
chmod 600 id_ed25519
ssh-add id_ed25519

# Copy the built files to the GitHub Pages repository
export SITE_HOME="$HOME/site"
git clone --quiet --depth 1 "git@github.com:university-map/university-map.github.io.git" "$SITE_HOME"
cp -rf dist/* "$SITE_HOME"

# Check if there are changes to commit and deploy
cd "$SITE_HOME"
if [[ -n $(git status -s) ]]; then
  git add -A
  git commit -m "[GitHub Actions] Deploy university-map/university-map@$GITHUB_SHA"
  git push -q
  echo "Finished Deployment"
else
  echo "No changes to deploy. Exiting with status 0."
  exit 0
fi
```

Finally, I added a search bar allowing users to search for keywords from a drop-down list or through commonly used acronyms such as "CMU" to find schools. Here's the corresponding commit: [88b1bb4](https://github.com/university-map/university-map/tree/88b1bb4959f8cd6ab5d7894c5ed1dd1d034ceee2).

![image4](https://github.com/university-map/university-map/assets/46962923/54dfdaa2-f874-47a0-a6af-f1ec2afd1b4b)

## What's Next?

With the project reaching the first milestone, the journey is far from over. Here are some exciting next steps I'm considering:
- Expanding the database to include more universities and detailed information such as yearly rankings, colleges, and living and employment environments.
- Integrating social sharing options such as share and embedded HTML.
- Improve mobile device display.

That's it for now, folks. Thank you for reading! 

[Star University Map on GitHub](https://github.com/university-map/university-map). 🌍🎓🚀