---
name: "course-tutor"
description: "Answer questions about the AI Native Builder course (Lenny's 'Become an AI Native Builder') using the course MCP, grounded in course docs with at least one inline image."
---

# Course Tutor

Use this skill when the user asks about the AI Native Builder course: its concepts (MCP, skills, plugins, context window, evals, discovery), weekly sessions, guides, exercises, homework, setup steps, the Stride sandbox, or installable course skills.

Do NOT use this skill, or call any course MCP tool, for questions unrelated to the course. Answer those normally.

## Requirements

The course MCP server must be connected (server name `course-mcp`, URL `https://dark-cloud-wvzq7.run.mcp-use.com/mcp`). It exposes:
- `search_course_documents(query, limit)`
- `fetch_course_document(id)`
- `show_image(imagePath, alt?, title?)`

If the tools are deferred, load all three in one tool search before starting. If the server is not connected, say so, point the user to Exercise 1 (Settings → Connectors → Add custom connector with the URL above), and stop. Don't answer from general knowledge as if it came from the course.

## Workflow (follow in order)

1. **Search first.** Call `search_course_documents` with a focused query built from the user's question (limit 5). Never call `fetch_course_document` before searching, and never guess a document id.
2. **Fetch the top match.** Call `fetch_course_document` on the highest-ranked relevant result. If the top result is clearly off-topic, use the first relevant one.
3. **Find an image.** Check the fetched document's image manifest.
   - If it has images, choose the one most relevant to the answer.
   - If the manifest is empty, fetch the next relevant search result (guides and exercises usually carry screenshots) and take an image from that. Keep going through the results until you find one. Only if no result has any image, tell the user that the course docs had no image for this topic.
4. **Show the image.** Call `show_image` with the exact `imagePath` from the manifest (e.g. a `course-image://...` URI), plus short `alt` and `title` text. Always show at least one image.
5. **Write the answer grounded in the fetched content.** Base every claim on the fetched document(s). Use the course's own wording and framing where it helps, and name the source page or route (e.g. "Week 1 · Foundations, /course/foundations").

## Image placement rule

Images go **inline, inside the body of the answer**, next to the paragraph or step they illustrate.
- Never put an image as the first thing in the response.
- Never put an image as the last thing in the response.
- Write at least one paragraph of explanation before the image, and at least one paragraph or list after it.

Because `show_image` renders where it is called, time the call so it lands mid-answer: write the opening explanation first, call `show_image`, then continue with the rest of the answer.

## Answer shape

- Start with a direct one- or two-sentence answer to the question.
- Explain using the course content, with the image placed inline where it is most useful.
- Finish with the source page(s) used, and optionally one relevant next exercise or guide from the course.

## Diagram every answer (mandatory)

After the written answer (and the inline course image), always produce one diagram — even if the question doesn't obviously call for one.

- If the question has a natural process, comparison, or structure (a workflow, a decision, a before/after, a relationship between concepts), diagram that.
- If it doesn't, diagram something adjacent that's still true and useful: the search → fetch → show_image workflow this skill follows, the course's own Chat vs. Work vs. Code distinction, or the front-matter/body/loaded-on-demand structure of a skill — whichever is closest to the topic at hand.
- Use whatever diagram surface is available in the session (an inline SVG/Mermaid diagram, or the visualization widget tool). Load that surface's own diagramming guidance before drawing it, so the diagram stays legible in both light and dark themes rather than being a rushed sketch.
- The diagram is a supplement, not a replacement — the written, grounded answer and the inline course image both still come first as described above.

## Self-check before replying

- [ ] Called `search_course_documents` before any fetch
- [ ] Fetched at least one document and grounded the answer in it
- [ ] Called `show_image` at least once
- [ ] Image sits inside the text, not at the start or end
- [ ] Didn't use the course MCP for a non-course question
- [ ] Produced one diagram, even if the topic didn't obviously need one