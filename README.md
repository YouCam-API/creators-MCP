![YouCam MCP for Creators](assets/banner-creators.png)

[![CONSOLE](https://img.shields.io/badge/YOUCAM-3183FF?style=for-the-badge)](https://yce.perfectcorp.com/api-console/)
[![DOCUMENT](https://img.shields.io/badge/DOCUMENT-FF2D78?style=for-the-badge)](https://docs.perfectcorp.com/develop/mcp.md)

Official Perfect Corp [Model Context Protocol (MCP)](https://modelcontextprotocol.io/docs/2026-07-28/getting-started/intro) server for creative production. This server lets MCP clients like [Claude Desktop](https://claude.ai/download), [Cursor](https://www.cursor.com), [Github Copilot](https://github.com/features/copilot), [Codex](https://openai.com/codex) and others generate images and video from prompts, restore and upscale photos, cut and replace backgrounds, remove objects, swap faces, and produce studio-grade headshots.

The client handles request formatting, authentication and asynchronous polling. You add one entry to a config file and start prompting.

**Server URL:** `https://mcp-api-01.makeupar.com/mcp/creators`

---

## Quickstart with Claude Desktop

1. Get your API key from the [YouCam API Console](https://yce.perfectcorp.com/api-console/en/api-keys/).
2. Install [Node.js and npm](https://docs.npmjs.com/downloading-and-installing-node-js-and-npm).
3. Go to **Claude → Settings → Developer → Edit Config → `claude_desktop_config.json`** and include the following:

```json
{
  "mcpServers": {
    "youcam-creators": {
      "command": "npx",
      "args": [
        "-y",
        "mcp-remote",
        "https://mcp-api-01.makeupar.com/mcp/creators",
        "--header",
        "Authorization:${AUTH_HEADER}"
      ],
      "env": {
        "AUTH_HEADER": "Bearer YOUR_API_KEY"
      }
    }
  }
}
```

4. Quit Claude Desktop completely via **File → Exit** — closing the window is not enough — then reopen it.

---

## Other MCP clients

### Cursor

1. Open **Settings → Tools & MCP → Add Custom MCP**.
2. Add the following to `mcp.json`:

```json
{
  "mcpServers": {
    "youcam-creators": {
      "url": "https://mcp-api-01.makeupar.com/mcp/creators",
      "type": "http",
      "headers": {
        "Authorization": "Bearer YOUR_API_KEY"
      }
    }
  }
}
```

3. Back in **Settings → Tools & MCP**, enable the tools you want under *Installed MCP Servers*.

### Copilot in VS Code

Run `> MCP: Add Server`, choose **HTTP**, enter the server URL, name the server, and set the `Authorization` header.

Or run `> MCP: Open User Configuration` and add the following to `mcp.json`:

```json
{
  "servers": {
    "youcam-creators": {
      "url": "https://mcp-api-01.makeupar.com/mcp/creators",
      "type": "http",
      "headers": {
        "Authorization": "Bearer YOUR_API_KEY"
      }
    }
  }
}
```

### Any other client

Any client that supports remote MCP over HTTP works with the same three values: the URL, an `Authorization` header, and the literal `Bearer ` prefix in front of your key. Configuration is static — changing the key requires a client reload.

---

## What you can build

| Use case | What the server does |
| --- | --- |
| Product photography pipelines | Isolates the subject, replaces the background from a prompt or template, then upscales and sharpens |
| Headshots and profile imagery at scale | Turns casual selfies into studio portraits, business headshots or stylised avatars from template libraries |
| Social and short-form video | Text-to-video and image-to-video generation, style transfer, background replacement and HD enhancement |
| Photo restoration | Colorises black-and-white photos, recovers shadows, corrects colour, removes unwanted objects and people |
| Creative concepting | Text-to-image and image-to-image generation with reference images, plus context-aware outpainting to any ratio |

The full tool list with parameters is in the [MCP documentation](https://docs.perfectcorp.com/develop/mcp.md).

> **Heads-up:** avatar, studio, headshot, background change and video style transfer each have a matching `*_Templates` tool. Call it first to see the available IDs, then pass the one you want into the main tool.

---

## Example usage

**YouCam API units are consumed by these tools. Video generation and enhancement cost materially more than image tools.**

Try asking Claude:

- "Cut the background out of this product photo, drop it onto a clean studio gradient, then upscale it 2x — show me each stage"
- "Turn this selfie into a LinkedIn headshot. List the templates first, then run the two most conservative ones"
- "Colorise this 1950s family photo, fix the exposure, and remove the telephone pole on the left"
- "Animate this landscape still into a 5-second video with a slow push-in, then apply an anime style transfer and enhance it to HD"
- "Generate four square product-hero images from this prompt, then extend the best one to 16:9 for the banner"

---

## How tasks and file uploads work

Generation and editing tools are **asynchronous**, and video jobs take noticeably longer than image jobs. The flow your client runs on your behalf:

1. **Provide inputs.** `File_Upload` opens a drag-and-drop widget; `Get_Upload_API_Info` returns the endpoint for programmatic upload. Text-to-image and text-to-video need no upload at all.
2. **Run prerequisite steps where they exist.** Face swap pairs with a face detection tool to map source faces onto targets. Background change isolates the subject before compositing. Masked tools — object removal and inpainting, in both photo and video — need a greyscale mask.
3. **Start the task.** The tool returns a task identifier, not a finished asset.
4. **Poll for the result.** `Get_Running_Task_Status` returns the current state and, once complete, the result URLs. Expect several polls for video.
5. **Download the output.** Result URLs are hosted temporarily — fetch and store anything you need to keep.

> Result URLs returned by the server must be passed through unmodified. Do not rewrite, shorten or proxy them.

Tools chain well — background removal → background change → enhance, or image generation → outpainting → image-to-video. Describe the finished asset and most clients will sequence the calls for you.

---

## Pricing and usage

Each tool consumes **units** from your YouCam API balance. Cost varies by feature, resolution, duration and output count.

Ask before you build a flow — the `Get_Feature_Cost` tool answers directly:

> "Compare the unit cost of AI_Video_Generator_Text_to_Video against AI_Image_Generator_Text_to_Image."

Balance, top-ups and usage history live in the [YouCam API Console](https://yce.perfectcorp.com/api-console/).

---

## Troubleshooting

**Server does not appear in the client.** Validate your JSON, confirm the config path against your client's documentation, and fully restart the client.

**Claude Desktop still shows nothing after editing the config.** The window was closed but the process kept running. Quit via **File → Exit**.

**`401 Unauthorized`.** Check the header reads `Bearer YOUR_API_KEY` — the `Bearer ` prefix is required. If the key was rotated, update it and reload the client.

**Tools are listed but every call fails.** Your network cannot reach `mcp-api-01.makeupar.com`. Check firewall, VPN and proxy rules for that host.

**A video task appears stuck.** Video jobs simply take longer. Keep polling `Get_Running_Task_Status` with the task ID before assuming failure.

**The client times out mid-generation.** The client-side tool timeout is shorter than the job. Poll for status in a follow-up turn rather than waiting inside a single call.

**A masked edit leaves artefacts.** The mask is imprecise or not greyscale. Supply a clean greyscale mask covering the full target area plus a small margin.

**Face swap misses a face.** The detection step was skipped, or the face is too small or too angled. Run face detection first and confirm the mapping.

**Output looks soft after upscaling.** The source resolution is too low for the chosen factor. Step down to 2x, or enhance before running further edits.

**Insufficient units.** Check usage and top up in the [YouCam API Console](https://yce.perfectcorp.com/api-console/).

---

## Support

Questions, bug reports and feature requests: [YouCamOnlineEditor_API@perfectcorp.com](mailto:YouCamOnlineEditor_API@perfectcorp.com)

When reporting a problem, include your MCP client and version, the tool that failed, and the task ID if one was returned.

---

© Perfect Corp. YouCam is a trademark of Perfect Corp.
