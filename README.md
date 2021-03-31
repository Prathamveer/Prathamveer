## Hello there 👋 I am Pratham 🤠

- 👨‍💻 I’m currently automating real-world tasks 🌎
- 💻 I’m know JS, Java, React Native[obviously with React] and Python 🐍
- 🔌 I'm also doing Robotics 🤖
- ✉️ Find me at : [📬](mailto:prathamchahal@gmail.com) ⬅️
- 💡 Trying to become a successful Ethical Hacker 👾

Twitter - [@ChahalPratham](https://twitter.com/ChahalPratham)
<br/>
Instagram - [@chxhxl06](https://www.instagram.com/chxhxl_06/)
<br/>
Spotify - [@Pratham Chahal](https://open.spotify.com/user/zq4cvtlej38cg0cvmztf9wayq)

![My GitHub stats](https://github-readme-stats.vercel.app/api?username=Prathamveer&theme=outrun&show_icons=true)
<a href="https://github.com/Prathamveer">
  <img align="center" src="https://github-readme-stats.vercel.app/api/top-langs/?username=Prathamveer&layout=compact&hide=python&theme=material-palenight" />
</a>
const { request } = require("./utils");
const retryer = require("./retryer");
require("dotenv").config();

const fetcher = (variables, token) => {
  return request(
    {
      query: `
      query userInfo($login: String!) {
        user(login: $login) {
          repositories(isFork: false, first: 100) {
            nodes {
              languages(first: 1) {
                edges {
                  size
                  node {
                    color
                    name
                  }
                }
              }
            }
          }
        }
      }
      `,
      variables,
    },
    {
      Authorization: `bearer ${token}`,
    }
  );
};

async function fetchTopLanguages(username) {
  if (!username) throw Error("Invalid username");

  let res = await retryer(fetcher, { login: username });

  if (res.data.errors) {
    console.log(res.data.errors);
    throw Error(res.data.errors[0].message || "Could not fetch user");
  }

  let repoNodes = res.data.data.user.repositories.nodes;

  // TODO: perf improvement
  repoNodes = repoNodes
    .filter((node) => {
      return node.languages.edges.length > 0;
    })
    .sort((a, b) => {
      return b.languages.edges[0].size - a.languages.edges[0].size;
    })
    .map((node) => {
      return node.languages.edges[0];
    })
    .reduce((acc, prev) => {
      let langSize = prev.size;
      if (acc[prev.node.name] && prev.node.name === acc[prev.node.name].name) {
        langSize = prev.size + acc[prev.node.name].size;
      }

      return {
        ...acc,
        [prev.node.name]: {
          name: prev.node.name,
          color: prev.node.color,
          size: langSize,
        },
      };
    }, {});

  const topLangs = Object.keys(repoNodes)
    .slice(0, 5)
    .reduce((result, key) => {
      result[key] = repoNodes[key];
      return result;
    }, {});

  return topLangs;
}

module.exports = fetchTopLanguages;
