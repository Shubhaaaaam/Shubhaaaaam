<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width,initial-scale=1.0">
  <title>GitHub Stats Card</title>
  <style>
    body { font-family: Arial, sans-serif; background: #f5f5f5; color: #333; display: flex; justify-content: center; align-items: center; height: 100vh; margin: 0; }
    .card { background: #fff; border-radius: 10px; box-shadow: 0 4px 10px rgba(0,0,0,0.1); padding: 20px; width: 360px; }
    .profile { text-align: center; }
    .profile img { width: 80px; height: 80px; border-radius: 50%; }
    .profile h2 { margin: 10px 0 5px; font-size: 1.2em; }
    .profile p { margin: 0; font-size: 0.9em; color: #666; }
    .stats, .languages { margin-top: 20px; }
    .stats div, .languages div { display: flex; justify-content: space-between; padding: 4px 0; }
    .languages-bar { height: 6px; border-radius: 4px; background: #ddd; overflow: hidden; margin-top: 4px; }
    .lang-bar-fill { height: 100%; background: #4CAF50; }
  </style>
</head>
<body>
  <div class="card">
    <div class="profile">
      <img id="avatar" src="" alt="Avatar">
      <h2 id="name">Username</h2>
      <p id="bio"></p>
    </div>
    <div class="stats">
      <div><span>Repos:</span><span id="repo-count">0</span></div>
      <div><span>Stars:</span><span id="star-count">0</span></div>
      <div><span>Followers:</span><span id="followers">0</span></div>
      <div><span>Following:</span><span id="following">0</span></div>
    </div>
    <div class="languages">
      <h3>Top Languages</h3>
      <div id="languages-list"></div>
    </div>
  </div>

  <script>
    const username = 'YOUR_GITHUB_USERNAME';

    async function fetchJSON(url) {
      const res = await fetch(url);
      if (!res.ok) throw new Error(`Failed to fetch ${url}`);
      return res.json();
    }

    async function loadStats() {
      const user = await fetchJSON(`https://api.github.com/users/${username}`);
      document.getElementById('avatar').src       = user.avatar_url;
      document.getElementById('name').textContent  = user.login;
      document.getElementById('bio').textContent   = user.bio || '';
      document.getElementById('repo-count').textContent = user.public_repos;
      document.getElementById('followers').textContent  = user.followers;
      document.getElementById('following').textContent  = user.following;

      // repos and stars
      let page = 1, repos = [];
      while (true) {
        const batch = await fetchJSON(`https://api.github.com/users/${username}/repos?per_page=100&page=${page}`);
        if (!batch.length) break;
        repos = repos.concat(batch);
        page++;
      }
      const totalStars = repos.reduce((sum, r) => sum + r.stargazers_count, 0);
      document.getElementById('star-count').textContent = totalStars;

      // top languages
      const langCount = {};
      repos.forEach(r => { if (r.language) langCount[r.language] = (langCount[r.language]||0) + 1; });
      const sorted = Object.entries(langCount).sort((a,b)=>b[1]-a[1]).slice(0,5);
      const max = sorted[0]?.[1]||1;

      const list = document.getElementById('languages-list');
      sorted.forEach(([lang,count]) => {
        const pct = Math.round((count/max)*100);
        const div = document.createElement('div');
        div.innerHTML = `
          <span>${lang}</span><span>${pct}%</span>
          <div class="languages-bar">
            <div class="lang-bar-fill" style="width:${pct}%"></div>
          </div>`;
        list.appendChild(div);
      });
    }

    loadStats().catch(console.error);
  </script>
</body>
</html>
