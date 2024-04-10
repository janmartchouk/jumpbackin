<script>
  let today = new Date();
  let yesterday = new Date().setDate(today.getDate() - 1);
  let startOfDay = 'a';
  // set start and end to today 4am and 4am tomorrow
  let start = new Date(new Date(yesterday).setHours(4, 0, 0, 0)).toISOString();
  let end = new Date(new Date(today).setHours(28, 0, 0, 0)).toISOString();

  let sites = [

  ]

  // defaults
  if (!localStorage.content || localStorage.content === '') {
    localStorage.content = JSON.stringify({
      pins: [],
      threshold: 5,
      watcher: 'aw-watcher-web-firefox'
    });
  }

  function getFromLSC(key) {
    return JSON.parse(localStorage.content)[key];
  }

  function writeToLSC(key, value) {
    let content = JSON.parse(localStorage.content);
    content[key] = value;
    localStorage.content = JSON.stringify(content);
  }

  let threshold = getFromLSC('threshold');
  let watcher = getFromLSC('watcher');

  $: writeToLSC('threshold', threshold);
  $: writeToLSC('watcher', watcher);

  let display_settings = false;
  $: display_settings_css = display_settings ? 'flex' : 'none';
  
  let classes = new Set([]);
  fetch('http://localhost:5600/api/0/settings/classes').then(r => r.json()).then(data => {
    for (const c of Object.values(data)) {
      if(c.rule.regex) {
        c.rule.regex = new RegExp(c.rule.regex.toLowerCase());
        classes.add(c);
      }
    }
  });

  function addNewClasses(classes, site) {
    for (const c of classes) {
          if (c.rule.regex.test(site.name)) {
            for (const cl of c.name) {
              site.classes.add(cl);
            }
          }
        }
  }

  fetch(`http://localhost:5600/api/0/buckets/${watcher}/events`).then(r => r.json()).then(data => {
    for (let event of data) {
      let site_exists = false;
      for (const site of sites) {
        if (site.name === new URL(event.data.url).host) {
          site_exists = true;
          site.duration = Math.round((event.duration/60) + site.duration);
        }
        addNewClasses(classes, site);
      }
      if (!site_exists) {
        let site = {
          name: new URL(event.data.url).host,
          duration: Math.round(event.duration /60),
          classes: new Set([]),
          pinned: false,
        };
        if(localStorage.content && localStorage.content.includes(site.name)) {
          site.pinned = true;
        }
        addNewClasses(classes, site);
        if(site.name) {sites = [...sites, site]};
      }
    }
  });

  $: sites.sort((a, b) => b.duration - a.duration);
  // put pinned ones first
  $: sites.sort((a, b) => b.pinned - a.pinned);

  function handlePin(site, e) {
    e.preventDefault();
    site.pinned = !site.pinned;
    if (site.pinned) {
      let pins = getFromLSC('pins');
      pins.push(site.name);
      writeToLSC('pins', pins);
    } else {
      let pins = getFromLSC('pins');
      pins = pins.filter(p => p !== site.name);
      writeToLSC('pins', pins);
    }
    sites = sites.sort((a, b) => b.pinned - a.pinned);
  }
</script>

<main>
    <!-- svelte-ignore a11y-click-events-have-key-events -->
    <!-- svelte-ignore a11y-no-static-element-interactions -->
    <div style='display:{display_settings_css}' class="settings" on:click={() => {display_settings = !display_settings}}>
      <!-- svelte-ignore a11y-no-static-element-interactions -->
      <div class="settings-content" on:click={(e) => {e.stopPropagation()}}>
        <h1 class='settings-title'>Settings</h1>
        <group>
          <label for='threshold'>Time Threshold: </label>
          <input type='number' id='threshold' min=0 bind:value={threshold}/>
        </group>
        <group>
          <label for='watcher'>Watcher: </label>
          <input type='text' id='watcher' bind:value={watcher}/>
        </group>
      </div>
    </div>
  <div class="content">
    <!-- svelte-ignore a11y-click-events-have-key-events -->
    <!-- svelte-ignore a11y-no-static-element-interactions -->
    <span class="settings-button" on:click={() => {display_settings = !display_settings}}>
      <svg xmlns="http://www.w3.org/2000/svg" height="16px" viewBox="0 0 16 16" width="16px"><path d="m 7.5 1.019531 c -0.550781 0 -0.996094 0.445313 -0.996094 0.996094 v 0.453125 c -0.472656 0.128906 -0.929687 0.320312 -1.355468 0.566406 l -0.324219 -0.324218 c -0.390625 -0.390626 -1.019531 -0.390626 -1.410157 0 l -0.703124 0.707031 c -0.390626 0.390625 -0.390626 1.019531 0 1.410156 l 0.320312 0.320313 c -0.246094 0.425781 -0.433594 0.882812 -0.5625 1.355468 h -0.453125 c -0.550781 0 -0.996094 0.445313 -0.996094 0.996094 v 1 c 0 0.550781 0.445313 0.996094 0.996094 0.996094 h 0.449219 c 0.132812 0.472656 0.320312 0.929687 0.566406 1.355468 l -0.320312 0.320313 c -0.390626 0.390625 -0.390626 1.019531 0 1.410156 l 0.703124 0.707031 c 0.390626 0.390626 1.019532 0.390626 1.410157 0 l 0.320312 -0.320312 c 0.429688 0.242188 0.882813 0.433594 1.359375 0.558594 v 0.457031 c 0 0.550781 0.445313 0.996094 0.996094 0.996094 h 0.996094 c 0.554687 0 1 -0.445313 1 -0.996094 v -0.453125 c 0.472656 -0.128906 0.929687 -0.320312 1.355468 -0.566406 l 0.320313 0.324218 c 0.390625 0.390626 1.019531 0.390626 1.410156 0 l 0.707031 -0.707031 c 0.390626 -0.390625 0.390626 -1.019531 0 -1.410156 l -0.320312 -0.320313 c 0.242188 -0.425781 0.433594 -0.882812 0.558594 -1.355468 h 0.453125 c 0.554687 0 1 -0.445313 1 -0.996094 v -1 c 0 -0.550781 -0.445313 -0.996094 -1 -0.996094 h -0.449219 c -0.128906 -0.472656 -0.320312 -0.929687 -0.566406 -1.355468 l 0.324218 -0.320313 c 0.390626 -0.390625 0.390626 -1.019531 0 -1.410156 l -0.707031 -0.707031 c -0.390625 -0.390626 -1.019531 -0.390626 -1.410156 0 l -0.320313 0.320312 c -0.425781 -0.242188 -0.882812 -0.429688 -1.355468 -0.558594 v -0.457031 c 0 -0.550781 -0.445313 -0.996094 -1 -0.996094 z m 0.515625 3.976563 c 1.660156 0 3 1.34375 3 3 s -1.339844 3 -3 3 c -1.65625 0 -3 -1.34375 -3 -3 s 1.34375 -3 3 -3 z m 0 0" fill="#222222"/></svg>
    </span>
    <h1>Jump back in</h1>
    <h2 id='info'>Left-click to open, right-click to pin.</h2>
    <div class="sites">
      {#each sites as site}
      {#if site.duration > threshold}
      <a on:contextmenu={(e) => {handlePin(site, e)}} href='https://{site.name}' class='site'>
        {#if site.pinned}
        <span class='pin'>
          <svg xmlns="http://www.w3.org/2000/svg" height="16px" viewBox="0 0 16 16" width="16px"><g fill="#222222"><path d="m 9 10 h -2 v 2 l 1 1 l 1 -1 z m 0 0"/><path d="m 10.707031 3.292969 c -0.1875 -0.1875 -0.441406 -0.292969 -0.707031 -0.292969 h -4 c -0.554688 0 -1 0.449219 -1 1 s 0.445312 1 1 1 h 4 c 0.550781 0 1 -0.449219 1 -1 c 0 -0.265625 -0.105469 -0.519531 -0.292969 -0.707031 z m 0 0"/><path d="m 4 8.988281 c 0 -2.207031 1.792969 -4 4 -4 s 4 1.792969 4 4 z m 0 0"/><path d="m 6 2.972656 l 0.222656 4.773438 h 3.585938 l 0.191406 -4.738282 z m 0 0"/></g></svg>
        </span>  
        {/if}
        <img alt='favicon' class='site-img' src={`https://icons.duckduckgo.com/ip3/${site.name}.ico`} width=128px/>
        <span class=site-right>
          <h2 class='site-name'>{site.name}</h2>
          <span class='site-bottom-line'>
            <p class='site-duration'>{site.duration}m</p>
            <p class='site-classes'>{Array.from(site.classes).join(', ')}</p>
          </span>
        </span>
      </a>
      {/if}
    {/each}
    </div>
  </div>
</main>