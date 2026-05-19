<template>
  <div id="app">
    <router-view />
    <div class="nav">
      <router-link to="/">记录</router-link>
      <router-link to="/timeline">时间线</router-link>
      <router-link to="/about">关于</router-link>
    </div>
  </div>
</template>

<style>
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700;800;900&display=swap');

:root {
  --bg: #f8f6f3;
  --card: #ffffff;
  --text: #1a1a1e;
  --text2: #6b6b76;
  --text3: #9a9aa6;
  --line: #e8e6e1;
  --soft: #f0eee9;
  --accent: #5b5bd6;
  --c1: #5b5bd6;
  --c2: #3b8fe0;
  --c3: #4cb782;
  --c4: #d4a04a;
  --c5: #e07b5b;
  --c6: #b877c9;
  --c7: #6bb5b5;
  --c8: #9a9aa6;
  --c9: #f06292;
  --c10: #7e57c2;
  --c11: #26c6da;
  --c12: #ff8a65;
  --c13: #aed581;
  --c14: #ffd54f;
  --c15: #ef5350;
  --c16: #78909c;
  --shadow: 0 1px 3px rgba(0, 0, 0, 0.04), 0 1px 2px rgba(0, 0, 0, 0.02);
  --shadow-lg: 0 8px 32px rgba(0, 0, 0, 0.06);
  --radius: 16px;
  --radius-sm: 10px;
}

html.dark {
  --bg: #121214;
  --card: #1e1e22;
  --text: #f0f0f2;
  --text2: #9a9aa6;
  --text3: #6b6b76;
  --line: #2a2a2e;
  --soft: #252529;
  --accent: #7c7ce0;
  --shadow: 0 1px 3px rgba(0, 0, 0, 0.2);
  --shadow-lg: 0 8px 32px rgba(0, 0, 0, 0.3);
}

* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

html,
body {
  height: 100%;
  font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'SF Pro Display', sans-serif;
  background: var(--bg);
  color: var(--text);
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

#app {
  max-width: 420px;
  margin: 0 auto;
  min-height: 100vh;
  background: var(--bg);
  display: flex;
  flex-direction: column;
  position: relative;
}

.nav {
  position: fixed;
  bottom: 0;
  left: 50%;
  transform: translateX(-50%);
  width: 100%;
  max-width: 420px;
  background: color-mix(in srgb, var(--card) 88%, transparent);
  backdrop-filter: blur(24px);
  -webkit-backdrop-filter: blur(24px);
  display: flex;
  padding: 6px 16px 0;
  z-index: 100;
  gap: 2px;
}

.nav a {
  flex: 1;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 3px;
  background: none;
  border: none;
  cursor: pointer;
  color: var(--text3);
  padding: 8px 0;
  text-decoration: none;
  border-radius: 10px;
  font-size: 11px;
  font-weight: 600;
  transition: all .2s;
}

.nav a:hover {
  background: var(--soft);
}

.nav a.router-link-exact-active {
  color: var(--accent);
  background: color-mix(in srgb, var(--accent) 8%, transparent);
}

@media(min-width:768px) {
  #app {
    margin: 20px auto;
    min-height: 90vh;
    overflow: hidden;
    box-shadow: var(--shadow-lg);
  }
}
</style>
