## Engine invariants — ห้ามละเมิดโดยไม่ถาม

engine = ฟังก์ชันบริสุทธิ์: CSV เข้า → record ออก

- ห้าม AI/LLM call, ห้าม network, ห้าม state/config file
- ห้าม tune/generate strategy — engine ต้องไม่มี stake ในผลลัพธ์
- ห้ามเพิ่ม check ใหม่จนกว่ามี user จริงขอพร้อมเคสตัวอย่าง
- ห้ามแตะ human report format เดิม (คนใช้อยู่แล้ว)
- fail-safe เข้าข้าง "ไม่รู้" มากกว่าเดาแล้วมั่นใจผิด

เหตุผลเต็ม: ROADMAP-TEMP.md · แผนปัจจุบัน: `.fapony/plan/`

<!-- code-review-graph: ถอดจาก MCP แล้ว (repo เล็ก ไม่คุ้ม context) —
     เรียกผ่าน CLI ได้เมื่อต้องการ ดู `code-review-graph --help`;
     เอากลับมาเป็น MCP ตอนเริ่ม cloud-v0 -->

## Memory — `fael` (append-only)

Rows live in the clone-shared journal (`.git/fael/`, `store = "local"`, the
default): every worktree sees them at once, nothing to commit. This repo is
public — never point `fael.remote` at origin (the rows would be published).

Log as you work — do not wait to be asked. Use the fael MCP tools when
connected, otherwise the CLI:

    fael kickoff <plan.md>            # start a session with this
    fael add decision "what was locked, and why" --files src/x.py
    fael add issue "what is broken" --files src/x.py
    fael add note "state the next session needs" --files src/x.py
    fael close <id> "fixed in <sha>"
    fael find "<text>"

What goes in — would someone picking this up tomorrow need it, and can they not
find it anywhere else?
- issue — something broken, even when found mid-task on something else: add it
  now, not at the end. Once fixed, close it.
- decision — something agreed or locked that git and the plan do not say, with why.
- note — state the next session needs (where a chunk stopped, what is half-done).
Before ending a turn that committed work: at least one row about it.

Write each entry standalone — it is read months later with no chat to refer to.
--files is required: rows that name no file cannot be recalled when that file is
touched later. Never type an id from memory — `fael find` it first.

## Executing a plan chunk-by-chunk

A long plan run in one unbroken session accumulates context with nothing to shrink
it — token cost and coherence both degrade with session length, not with amount of
work done. Cut at chunk boundaries instead:

Finish a chunk, before starting the next:
1. Tick its checkbox + stamp the TL;DR in the plan file
2. Commit — separate from other chunks
3. `fael add note "what the next chunk needs" --files f1,f2,plan:<name> --key plan:<name>:handoff`
   — one key per plan, so each chunk's note supersedes the last
4. Stop. Do not continue to the next chunk in the same session unless told to.

Next chunk, new session — open with `fael kickoff <path/to/PLAN-x.md>` instead
of carrying the old transcript forward. kickoff already filters to the rows for that plan,
and takes just the filename (`kickoff PLAN-x.md`) when you do not want to type the path.
