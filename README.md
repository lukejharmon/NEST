
# NEST - Network for East Africa Science and Training
<img style="float: left;" src="images/nest_logo2.png" alt="NEST" width="400"/>

About NEST

Find committee members

Collaboration network



NEST is a group of faculty interested in fostering scientific collaborations in bioinformatics with institutions in East Africa.

1. People and institutions
2. Find potential committee members
3. Collaboration network

## People and institutions

### University of Idaho

Onesmo Balemba

Christine Parent

Luke Harmon

### Virginia Tech University

Josef Uyeda

Elizabeth Daniel

### Nelson Mandela Institute of African Technology

xxx


<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>NEST Faculty Finder</title>
  <!-- Tailwind (CDN) -->
  <script src="https://cdn.tailwindcss.com"></script>
  <!-- Fuse.js for fuzzy search -->
  <script src="https://cdn.jsdelivr.net/npm/fuse.js@6.6.2"></script>
</head>
<body class="bg-neutral-50 text-neutral-900">
  <header class="max-w-5xl mx-auto px-4 py-8">
    <h1 class="text-3xl font-extrabold tracking-tight">NEST Faculty Finder</h1>
    <p class="text-sm text-neutral-600 mt-1">
      Search by research keywords (e.g., <em>bioinformatics, genomics, phylogenetics</em>) to find collaborators across East Africa and partner institutions.
    </p>
  </header>

  <main class="max-w-5xl mx-auto px-4 pb-16">
    <!-- Search + filters -->
    <div class="bg-white rounded-2xl shadow p-4 mb-6">
      <div class="flex flex-col gap-3 md:flex-row md:items-center">
        <input id="q" type="search" inputmode="search" placeholder="Search by keywords, topics, institution, country, name…"
               class="w-full border border-neutral-300 rounded-xl px-4 py-3 focus:outline-none focus:ring-2 focus:ring-emerald-500"
               aria-label="Keyword search" />
        <select id="countryFilter" class="border border-neutral-300 rounded-xl px-3 py-3 focus:outline-none focus:ring-2 focus:ring-emerald-500 w-full md:w-64" aria-label="Country filter">
          <option value="">All countries</option>
        </select>
      </div>
      <div class="flex items-center gap-3 mt-3">
        <label class="inline-flex items-center gap-2 text-sm">
          <input id="exactMatch" type="checkbox" class="w-4 h-4 rounded border-neutral-300">
          Exact match
        </label>
        <button id="clearBtn" class="ml-auto text-sm underline underline-offset-4">Clear</button>
      </div>
    </div>

    <!-- Results summary -->
    <div id="summary" class="text-sm text-neutral-600 mb-3" aria-live="polite"></div>

    <!-- Results grid -->
    <div id="results" class="grid sm:grid-cols-2 lg:grid-cols-3 gap-4"></div>

    <!-- Empty state -->
    <template id="emptyTpl">
      <div class="col-span-full bg-white rounded-2xl p-10 text-center border border-dashed border-neutral-300">
        <p class="text-lg font-semibold">No matches found</p>
        <p class="text-sm text-neutral-600 mt-1">Try different keywords or clear filters.</p>
      </div>
    </template>

    <!-- Card template -->
    <template id="cardTpl">
      <article class="bg-white rounded-2xl shadow hover:shadow-md transition p-5 flex flex-col gap-2">
        <h2 class="text-lg font-bold"></h2>
        <p class="text-sm text-neutral-600"></p>
        <p class="text-sm"></p>
        <div class="flex flex-wrap gap-2 mt-2"></div>
        <div class="mt-3 text-sm text-neutral-600"></div>
      </article>
    </template>
  </main>

  <footer class="max-w-5xl mx-auto px-4 pb-10 text-xs text-neutral-500">
    NEST — Network for East Africa Science &amp; Training
  </footer>

  <script>
    // Configuration for Fuse.js: weights reflect how we want to score matches.
    const fuseOptionsBase = {
      includeScore: true,
      threshold: 0.35,     // lower = stricter
      ignoreLocation: true,
      minMatchCharLength: 2,
      keys: [
        { name: 'name', weight: 0.3 },
        { name: 'institution', weight: 0.2 },
        { name: 'country', weight: 0.2 },
        { name: 'topics', weight: 0.3 },
        { name: 'keywords', weight: 0.5 } // arrays are supported
      ]
    };

    const el = {
      q: document.getElementById('q'),
      results: document.getElementById('results'),
      summary: document.getElementById('summary'),
      countryFilter: document.getElementById('countryFilter'),
      exactMatch: document.getElementById('exactMatch'),
      cardTpl: document.getElementById('cardTpl'),
      emptyTpl: document.getElementById('emptyTpl'),
      clearBtn: document.getElementById('clearBtn')
    };

    let faculty = [];
    let fuse;

    async function loadData() {
      const res = await fetch('faculty.json', { cache: 'no-cache' });
      faculty = await res.json();

      // Populate country filter (unique, sorted)
      const countries = Array.from(new Set(faculty.map(f => (f.country || '').trim()).filter(Boolean))).sort();
      for (const c of countries) {
        const opt = document.createElement('option');
        opt.value = c;
        opt.textContent = c;
        el.countryFilter.appendChild(opt);
      }

      initFuse();
      render();
    }

    function initFuse() {
      const opts = { ...fuseOptionsBase };
      // Exact match toggle: make it stricter by lowering threshold and using extended search
      if (el.exactMatch.checked && el.q.value.trim()) {
        opts.threshold = 0.2;
        fuse = new Fuse(faculty, opts);
      } else {
        fuse = new Fuse(faculty, opts);
      }
    }

    function queryData() {
      let data = faculty.slice();
      const q = el.q.value.trim();
      const country = el.countryFilter.value;

      if (q) {
        // For "exact match", try Fuse extended search with "=" per token
        if (el.exactMatch.checked) {
          const tokens = q.split(/\s+/).map(tok => `="${tok}"`);
          const pattern = tokens.join(' ');
          data = fuse.search(pattern).map(r => r.item);
        } else {
          data = fuse.search(q).map(r => r.item);
        }
      }

      if (country) {
        data = data.filter(d => (d.country || '').toLowerCase() === country.toLowerCase());
      }

      return data;
    }

    function render() {
      const data = queryData();
      el.results.innerHTML = '';

      if (data.length === 0) {
        el.summary.textContent = '0 results';
        el.results.appendChild(el.emptyTpl.content.cloneNode(true));
        return;
      }

      el.summary.textContent = `${data.length} ${data.length === 1 ? 'result' : 'results'}`;

      for (const f of data) {
        const card = el.cardTpl.content.cloneNode(true);
        const [h2, metaP, topicsP, chipsDiv, contactDiv] = card.querySelectorAll('h2, p, div');

        h2.textContent = f.name || 'Unknown';
        metaP.textContent = [f.institution, f.country].filter(Boolean).join(' — ');
        topicsP.textContent = f.topics || '';

        // keyword chips
        chipsDiv.setAttribute('aria-label', 'Keywords');
        (f.keywords || []).forEach(k => {
          const span = document.createElement('span');
          span.className = 'text-xs bg-emerald-50 border border-emerald-200 text-emerald-700 px-2 py-1 rounded-full';
          span.textContent = k;
          chipsDiv.appendChild(span);
        });

        // contact / links
        const bits = [];
        if (f.email) bits.push(`<a class="underline" href="mailto:${f.email}">${f.email}</a>`);
        if (f.website) bits.push(`<a class="underline" href="${f.website}" target="_blank" rel="noopener">Website</a>`);
        contactDiv.innerHTML = bits.join(' · ');

        el.results.appendChild(card);
      }
    }

    // Events
    el.q.addEventListener('input', () => render());
    el.countryFilter.addEventListener('change', () => render());
    el.exactMatch.addEventListener('change', () => { initFuse(); render(); });
    el.clearBtn.addEventListener('click', () => {
      el.q.value = '';
      el.countryFilter.value = '';
      el.exactMatch.checked = false;
      initFuse();
      render();
      el.q.focus();
    });

    loadData().catch(err => {
      el.summary.textContent = 'Error loading data.';
      console.error(err);
    });
  </script>
</body>
</html>

## Collaboration Network

(webapp here)
