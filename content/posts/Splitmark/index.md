---
title: Building Splitmark - A CLI-First Markdown Editor with Cloud Sync
date: "2025-10-14T15:40:32.169Z"
template: "post"
draft: false
slug: "building-splitmark"
category: "Node.js"
tags:
  - "Terminal Markdown Editor"
  - "Software Architecture"
  - "SaaS"
description: "Building Splitmark - A CLI-First Markdown Editor with Cloud Sync"
---

## The Problem That Started It All

Picture this: You're deep in a coding session, multiple terminal tabs open, when your manager pings you about an urgent meeting in 5 minutes. You need to take notes, but opening Notion feels like overkill and breaks your flow. VS Code is already running your project. You just want something fast, lightweight, and native to your terminal workflow.

This exact scenario happened to me countless times over the past year. As a developer who spends 80% of my day in the terminal, I found myself constantly context-switching just to jot down quick thoughts or meeting notes. The existing solutions either felt too heavy (Electron apps), too disconnected from my workflow (web apps), or too limited (basic CLI tools).

That's when I decided to build Splitmark - a CLI-first markdown editor that doesn't compromise on either speed or functionality.

## Design Philosophy: CLI-First, Cloud-Enhanced

From day one, I knew this had to be a CLI-native experience. No Electron, no web-wrapper disguised as a desktop app. Just pure, fast terminal goodness. But I also recognized that pure CLI tools have limitations - what about mobile access? What about sharing with non-technical team members?

The solution was to build it as a **CLI-first, cloud-enhanced** platform:

1. **CLI as the primary interface** - Where developers spend their time
2. **Web as the universal interface** - For access anywhere, anytime
3. **Cloud sync as the bridge** - Seamless experience across contexts

## The Technical Stack: Choosing Speed and Flexibility

### CLI Application: Node.js + Ink

For the CLI, I chose **Node.js** despite the usual performance concerns. Why? Because the target audience (developers) already has Node installed, and the ecosystem is unmatched for rapid development. Plus, for text editing operations, the performance difference vs. Rust or Go is negligible in practice.

The real magic comes from **Ink** - React for the terminal. This was a game-changer because:

```javascript
// Clean, component-based terminal UI
function FileExplorer({ files, selectedIndex }) {
  return (
    <Box flexDirection="column">
      {files.map((file, index) => (
        <Text
          key={file.name}
          color={index === selectedIndex ? "blue" : "white"}
        >
          {file.name}
        </Text>
      ))}
    </Box>
  );
}
```

No more wrestling with terminal control sequences or cursor positioning. Just React components that render to the terminal. The development velocity this enabled was incredible.

### Cloud Infrastructure: Cloudflare Workers + D1

For the backend, I went all-in on **Cloudflare's edge stack**:

- **Cloudflare Workers** for the API
- **D1 SQLite** for the database
- **R2** for file storage
- **Zero cold starts** and **global edge deployment**

This wasn't just about performance (though <50ms response times globally are nice). Cloudflare's integrated stack meant I could focus on features instead of DevOps. No Kubernetes clusters, no database scaling concerns, no CDN configuration.

```javascript
// Workers API endpoint - deploys to 300+ edge locations
export default {
  async fetch(request, env) {
    const response = await handleApiRequest(request, env.DB, env.R2);
    return new Response(JSON.stringify(response), {
      headers: { "Content-Type": "application/json" },
    });
  },
};
```

The entire backend infrastructure is defined in code and deploys in seconds. When you're a solo developer building a side project, this kind of simplicity is invaluable.

### Web Frontend: React 19 + Material-UI

For the web interface, I chose **React 19** to experiment with the new features:

- **Server Components** for initial page loads
- **useActionState** for form handling
- **Automatic batching** for better performance

The web editor needed to feel fast and responsive, especially since users might be coming from a lightning-fast CLI experience. React 19's improvements to concurrent rendering and automatic optimizations helped achieve that seamless feel.

```javascript
// Using React 19's new form actions
function CreateFileForm() {
  const [isPending, formAction] = useActionState(createFileAction);

  return (
    <form action={formAction}>
      <input name="filename" placeholder="File name" />
      <button type="submit" disabled={isPending}>
        {isPending ? "Creating..." : "Create File"}
      </button>
    </form>
  );
}
```

## Engineering Challenges and Solutions

### Challenge 1: Real-Time Sync Without Complexity

The biggest technical challenge was syncing files between CLI and web without building a complex real-time system. WebSockets felt like overkill for what's essentially a personal note-taking tool.

**Solution: Smart Polling + Conflict Resolution**

I implemented a hybrid approach:

- **Optimistic updates** in both CLI and web
- **Smart polling** that backs off when no changes detected
- **Last-write-wins** with timestamp-based conflict resolution
- **Local caching** to work offline

```javascript
// Intelligent sync that adapts to usage patterns
async function syncFile(localFile, remoteVersion) {
  if (localFile.lastModified > remoteVersion.lastModified) {
    return await uploadFile(localFile);
  } else if (remoteVersion.lastModified > localFile.lastModified) {
    return await downloadFile(remoteVersion);
  }
  // Files are in sync
  return localFile;
}
```

This gives 95% of the benefits of real-time sync with 10% of the complexity.

### Challenge 2: Terminal Performance at Scale

As file counts grew during testing, the CLI's file explorer became sluggish. Terminal rendering is inherently slower than GUI rendering, so every optimization matters.

**Solution: Virtualization + Smart Caching**

I implemented terminal virtualization - only rendering visible items:

```javascript
// Only render visible files in the terminal viewport
function VirtualizedFileList({ files, viewportHeight }) {
  const visibleFiles = useMemo(() => {
    const start = Math.max(0, scrollTop - buffer);
    const end = Math.min(files.length, start + viewportHeight + buffer * 2);
    return files.slice(start, end);
  }, [files, scrollTop, viewportHeight]);

  return visibleFiles.map((file) => <FileItem key={file.id} file={file} />);
}
```

Plus aggressive caching of file metadata and lazy loading of file contents. The result? Smooth scrolling through thousands of files.

### Challenge 3: Cross-Platform CLI Distribution

Getting the CLI into developers' hands needed to be frictionless. npm was the obvious choice, but there were platform-specific gotchas around file permissions and PATH configuration.

**Solution: Smart Installation + Self-Healing**

The npm package includes post-install scripts that:

- Set correct file permissions across platforms
- Add shell completions for common shells
- Validate installation and provide helpful error messages

```bash
# Cross-platform installation that "just works"
npm install -g splitmark
splitmark --version  # Validates installation
splitmark init       # Sets up shell completions
```

## Security and Privacy: End-to-End Encryption

Since this tool handles potentially sensitive notes and documents, security was non-negotiable. I implemented **client-side encryption** where files are encrypted before leaving the user's device.

```javascript
// Files are encrypted client-side before upload
async function encryptFile(content, userKey) {
  const iv = crypto.getRandomValues(new Uint8Array(16));
  const encrypted = await crypto.subtle.encrypt(
    { name: "AES-GCM", iv },
    userKey,
    new TextEncoder().encode(content),
  );
  return { encrypted, iv };
}
```

The server never sees plaintext content. Even if Cloudflare's infrastructure were compromised, user files would remain encrypted. This was crucial for adoption among security-conscious developers.

## Performance Obsession: Measuring What Matters

Throughout development, I obsessively measured performance metrics that directly impact user experience:

### CLI Performance Metrics:

- **Startup time**: <100ms cold start
- **File search**: <50ms for 1000+ files
- **Sync operation**: <200ms for typical file
- **Memory usage**: <50MB for large projects

### Web Performance Metrics:

- **Time to Interactive**: 1.8s (React 19 improvements)
- **First Contentful Paint**: 0.7s
- **Largest Contentful Paint**: 1.2s
- **Bundle size**: 45.1kb gzipped

These weren't arbitrary goals - they were based on user expectations coming from other developer tools. VS Code starts in ~500ms, so Splitmark needed to be faster. GitHub loads in ~2s, so the web interface needed to beat that.

## User Research: Building for Real Developers

One particularly valuable insight: developers don't want another "productivity system" - they want tools that fit into their existing workflow. This reinforced the CLI-first approach.

## Development Velocity: Modern Tooling and Practices

To move fast as a solo developer, I invested heavily in development tooling:

### Testing Strategy:

- **Jest** with ES modules for comprehensive test coverage
- **Component testing** for CLI interfaces using Ink's test utilities
- **Integration tests** for sync functionality
- **Performance tests** to catch regressions

### Development Workflow:

- **ESLint + Prettier** for consistent code style
- **GitHub Actions** for CI/CD

### Deployment Pipeline:

- **Automatic testing** on every commit
- **Staging deployments** for feature branches
- **Blue-green deployments** for zero-downtime releases

```yaml
# GitHub Actions workflow for testing and deployment
name: CI/CD Pipeline
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Run tests
        run: npm test
      - name: Deploy to staging
        if: github.ref == 'refs/heads/develop'
        run: wrangler publish --env staging
```

This setup enabled me to ship features confidently and quickly iterate based on user feedback.

## Lessons Learned: What I'd Do Differently

### Technical Decisions:

- **Implement feature flags early** - Rolling back features is harder than toggling them off
- **Invest in observability sooner** - Understanding user behavior in production is crucial

### Product Decisions:

- **Talk to users earlier and more frequently** - Several features could have been avoided

### Development Process:

- **Automate deployment from day one** - Manual deployments don't scale
- **Write more integration tests** - Unit tests miss real-world usage patterns
- **Document architectural decisions** - Future me appreciated this

## The Future: Where Splitmark Goes Next

Based on user feedback and usage patterns, here's what's coming:

### Q1 2026 Roadmap:

- **Mobile app** for iOS/Android (React Native)
- **Plugin system** for community extensions
- **Team collaboration** features for shared workspaces
- **Advanced search** with full-text indexing

### Long-term Vision:

- **Blog Folders** - Most tech enthusiasts use markdown for blogging. I want to create a seamless experience for bloggers to manage their posts in Splitmark and publish directly to platforms like Dev.to, Hashnode, or even custom static site generators. For example, this post was written in Splitmark and published to my personal blog.

The goal remains the same: enhance the developer workflow without getting in the way.

## Conclusion: Building for Yourself First

Splitmark started as a personal frustration and became a tool used by thousands of developers. The key insight was building something I genuinely needed and used daily. When you're your own primary user, product decisions become clearer, and quality standards remain high.

The technical choices - CLI-first design, Cloudflare's edge stack, client-side encryption - weren't just about following trends. They were deliberate decisions to create the fastest, most reliable experience possible for the target audience.

Most importantly, the engineering process remained enjoyable throughout. Modern tooling, thoughtful architecture, and focusing on real user problems made building Splitmark feel less like work and more like solving interesting puzzles.

If you're considering building a developer tool, my advice is simple: start with your own pain point, choose boring technology where possible, and obsess over the fundamentals - performance, reliability, and user experience. The rest follows naturally.

---

**Try Splitmark:**

- CLI: `npm install -g splitmark`
- Web: [https://splitmark.app](https://splitmark.app)
- GitHub: [https://github.com/splitmark/splitmark](https://github.com/splitmark/splitmark)

---

_This post was written in Splitmark's CLI editor and polished in the web interface - a perfect example of the workflow the tool enables._
