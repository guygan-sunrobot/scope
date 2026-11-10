---
name: scope
description: เปิด claude.ai connector เฉพาะที่งานปัจจุบันต้องใช้ (ตอนนี้คือ Google Drive และ Claude Docs) ทำงานอัตโนมัติผ่าน hooks; เรียกเองเมื่อผู้ใช้พูดว่า scope/เปิดเฉพาะที่จำเป็น/ลด token หรือเพื่อตรวจว่า token หมดไปกับอะไร
---

# scope

ตอบผู้ใช้เป็นภาษาไทย

connector ที่เปิดอยู่ทุกตัวจะเพิ่มชื่อ tool เข้าไปในทุก request ค่าเริ่มต้นจึงแคบ: เปิดเฉพาะ `always` (ว่างไว้) ที่เหลือเปิดเมื่อข้อความต้องใช้

## อัตโนมัติ (hooks)

- `UserPromptSubmit` รัน `scope hook-prompt` เงียบไว้ เว้นแต่ข้อความพูดถึงงานที่ใช้ connector (เช่น "ขึ้น Google Drive", "เปิด Claude Docs") จึงมีโน้ต `scope:` ขอให้เปิด connector นั้น เปิดเมื่อจำเป็นจริงเท่านั้น บอกผู้ใช้ว่ามีผลเทิร์นถัดไป แล้วหยุด
- เมื่อ context >= 100k และเพิ่งจบขั้นงาน (commit, test ผ่าน, แก้ STATUS.md, "โอเค/ok", เปลี่ยนหัวข้อ, ว่าง 30 นาที) หรือ context >= 300k ไม่ว่าจะเป็นอย่างไร จะมีโน้ต `scope:` ให้ปรับ `STATUS.md` ของโปรเจกต์หลังตอบข้อความนี้ แล้วบอกผู้ใช้ว่า `/clear` ได้ ผู้ใช้เป็นคน clear เอง
- `SessionStart` รัน `scope hook-start` ตอน startup, `/clear` และ compaction จะใส่ `STATUS.md` ของโปรเจกต์ (เฉพาะใต้ `status.trustedRoots` และตัดที่ `status.maxChars`; โฟลเดอร์อื่นใช้ `STATUS.md` ในโฟลเดอร์งานเอง เพราะ `status.inFolder` เปิดอยู่ (ถ้าปิด จะใช้ไฟล์ส่วนตัวใน `~/.local/share/scope/handoff/`) และคืนเฉพาะหลัง clear ภายใน 24 ชม.) ให้ทำต่อจากหัวข้อ `## Next` ถ้างานก่อนหน้าทิ้ง connector ไว้ จะขอให้ปิด เว้นแต่ข้อความแรกต้องใช้
- ปิดแต่ละฟีเจอร์ได้ที่ `/scope:setup` หรือ `scope features set <name>=on|off` ปิดทั้งหมด: `SCOPE_OFF=1`

การสลับ connector: โหลด `mcp__ccd_connectors__set_session_connector_enabled` ผ่าน ToolSearch เรียกทีละ connector พร้อมกันในชุดเดียว ผลมีเมื่อจบเทิร์นและเป็นค่าเริ่มต้นของ session ใหม่

## เรียกเอง

- `scope plan "<งาน>" [--profile a,b]`: งานนี้ต้องใช้ connector ใด (`want`) เทียบกับแถว `kind: "connector"` จาก `mcp__ccd_connectors__session_connectors_status` เปิดตัวที่ต้องใช้แต่ `disabled` ปิดตัวที่เหลือ
- `scope audit [--days 7]`: สถิติต่อ session จาก transcript (จำนวน request, context ฐาน, context สูงสุด)
- `scope profiles`: รายการ profile
- `scope features [set name=value ...]`: ดูหรือเปลี่ยนสวิตช์ฟีเจอร์

รัน CLI ด้วย `node "${CLAUDE_PLUGIN_ROOT}/bin/scope.mjs" <command>` ค่าเริ่มต้นอยู่ที่ `src/profiles.json` ของ plugin; ค่าส่วนตัว (`status.trustedRoots`, threshold, profile เพิ่มเติม) ใส่ที่ `~/.config/scope/config.json` ซึ่ง merge ทับและอยู่รอดตอนอัปเดต plugin

สลับได้เฉพาะแถว `kind: "connector"` (ตอนนี้มีแค่ Google Drive กับ Claude Docs) ส่วน plugin server (bio-research, engineering, data), server ใน `.mcp.json` ของโปรเจกต์ และ desktop extension จัดการที่ที่ตั้งค่าของมันเอง ห้ามแก้เงียบๆ
