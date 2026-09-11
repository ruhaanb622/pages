---
toc: false
layout: post
title: Capstone Project
description: Focused capstone project detail view
permalink: /capstone/view/
capstone:
  badge: Design-Based Research Capstone
  default_status: In Development
  cta_label: View Project
---

<div id="cv-root"></div>

<script>
(function () {
  const root = document.getElementById('cv-root');
  const id = new URLSearchParams(location.search).get('id');
  const pageDefaults = {
    title: {{ page.title | jsonify }},
    badge: {{ page.capstone.badge | jsonify }},
    status: {{ page.capstone.default_status | jsonify }},
    ctaLabel: {{ page.capstone.cta_label | jsonify }}
  };

  function esc(value) {
    return String(value || '')
      .replace(/&/g, '&amp;')
      .replace(/</g, '&lt;')
      .replace(/>/g, '&gt;')
      .replace(/"/g, '&quot;')
      .replace(/'/g, '&#39;');
  }

  function lines(value) {
    const values = Array.isArray(value) ? value : String(value || '').split('\n');
    return values.map(item => String(item).trim()).filter(Boolean);
  }

  function safeLink(value) {
    const raw = String(value || '').trim();
    if (!raw) return '';

    try {
      const url = new URL(raw, location.origin);
      return ['http:', 'https:'].includes(url.protocol) ? url.href : '';
    } catch (error) {
      console.warn('Ignoring invalid capstone project URL:', raw, error);
      return '';
    }
  }

  function showMessage(message, type = 'loading') {
    root.innerHTML = `<p class="capstone-message capstone-message--${type}">${esc(message)}</p>`;
  }

  if (!id) {
    showMessage('No project ID specified.', 'error');
    return;
  }

  // Local projects are available only for the current browser session.
  if (id.startsWith('local_')) {
    try {
      const all = JSON.parse(sessionStorage.getItem('ncLP') || '[]');
      const project = all.find(item => item.id === id);
      if (project) {
        renderProject(project);
        return;
      }
    } catch (error) {
      console.warn('Unable to read local capstone project data:', error);
    }

    showMessage('Project not found in this session.', 'error');
    return;
  }

  // Remote projects come from the capstone API.
  const API = window.javaURI || 'http://localhost:8585';
  showMessage('Loading project…');

  fetch(`${API}/api/capstones/${encodeURIComponent(id)}`, { credentials: 'include' })
    .then(response => {
      if (!response.ok) throw new Error(`HTTP ${response.status}`);
      return response.json();
    })
    .then(renderProject)
    .catch(error => {
      console.error('Unable to load capstone project:', error);
      showMessage('Could not load project. The server may be unavailable.', 'error');
    });

  function renderProject(project) {
    const title = project.title || pageDefaults.title;
    const tech = lines(project.tech);
    const keyPoints = lines(project.keyPoints);
    const impact = lines(project.impact);
    const team = lines(project.teamMembers);
    const link = safeLink(project.pageUrl);
    const course = String(project.courseCode || '').toUpperCase();
    const about = project.about || project.description || '';
    const hasHighlights = keyPoints.length > 0 || tech.length > 0;
    const hasDetails = Boolean(about || impact.length || link);

    document.title = `${title} — Capstone`;

    const image = project.imageUrl
      ? link
        ? `<a href="${esc(link)}" class="cv-image-link" target="_blank" rel="noopener">
             <img src="${esc(project.imageUrl)}" alt="${esc(title)}" class="cv-image">
             <div class="cv-overlay"><span>${esc(pageDefaults.ctaLabel)}</span></div>
           </a>`
        : `<div class="cv-image-link">
             <img src="${esc(project.imageUrl)}" alt="${esc(title)}" class="cv-image">
           </div>`
      : `<div class="cv-image-placeholder">${esc(title.slice(0, 3).toUpperCase())}</div>`;

    root.innerHTML = `
<div class="cv-infograph">
  <header class="cv-header">
    <span class="cv-badge">${esc(course || pageDefaults.badge)}</span>
    <h1 class="cv-title">${esc(title)}</h1>
  </header>

  <section class="cv-card" aria-label="${esc(title)} project details">
    <div class="ocs-grid ocs-grid--capstone">
      <article class="cv-visual">
        ${image}
        <div class="cv-status">${esc(project.status || pageDefaults.status)}</div>
        ${team.length ? `
        <div class="cv-team">
          <span class="cv-team-label">Project Leads</span>
          <span class="cv-team-name">${esc(team.join(', '))}</span>
        </div>` : ''}
      </article>

      ${hasHighlights ? `
      <section class="cv-content" aria-labelledby="cv-highlights-title">
        <h2 id="cv-highlights-title" class="cv-section-title">Highlights</h2>
        ${keyPoints.length ? `
        <div class="cv-keypoints">
          ${keyPoints.map(point => `<div class="cv-keypoint"><span class="cv-check">✓</span><span>${esc(point)}</span></div>`).join('')}
        </div>` : ''}
        ${tech.length ? `
        <div class="cv-tech-stack" aria-label="Technology used">
          ${tech.map(item => `<span class="cv-tech-tag">${esc(item)}</span>`).join('')}
        </div>` : ''}
      </section>` : ''}

      ${hasDetails ? `
      <section class="cv-details" aria-labelledby="cv-details-title">
        ${about ? `
        <h2 id="cv-details-title" class="cv-section-title">About</h2>
        <p class="cv-about">${esc(about)}</p>` : '<h2 id="cv-details-title" class="cv-section-title">Project</h2>'}
        ${impact.length ? `
        <h3 class="cv-section-title">Impact</h3>
        <div class="cv-impact-list">
          ${impact.map(item => `<div class="cv-impact-item">${esc(item)}</div>`).join('')}
        </div>` : ''}
        ${link ? `<a href="${esc(link)}" class="ocs-button ocs-button--primary" target="_blank" rel="noopener">${esc(pageDefaults.ctaLabel)}</a>` : ''}
      </section>` : ''}
    </div>
  </section>
</div>`;
  }
})();
</script>
