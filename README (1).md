# ตาที่สาม (Third Eye) — A Phone Sonar for Blind and Low-Vision Walkers

A web app that turns a phone camera into a **3-direction sonar with a range of up to about 8–10 m**. Hang the phone around your neck or hold it at chest height, and it beeps for obstacles on the **left, center and right**. The closer an obstacle is, the faster it beeps. **People, vehicles and objects each have their own sound.** The app also understands movement: it gives a **spoken warning in Thai** when something rushes toward you, cuts across your path, or is blocking your way within 2 m.

**Live demo:** https://sippakornq.github.io/third-eye/ (open on a phone)

## The problem
Thai sidewalks are crowded with parked and moving motorcycles, vendor stalls, tables, chairs and people. A white cane only detects what it touches, about one step ahead and at ground level. It cannot tell a blind walker that a motorcycle is speeding toward them, that a car is about to cross in front of them, or what exactly is blocking the way. Without help, blind and low-vision people have to walk slowly, bump into things, or depend on someone to guide them.

## Who it is for
Blind and low-vision people walking on sidewalks, around campus, or around their dorm. It is a helper to use **together with a white cane**, not a replacement for one.

## What you hear

| Sound | Meaning |
|---|---|
| Soft, slow "pok … pok …" | Path clear: the system is running and nothing is within range |
| High clear "tid-tid" | A **person** |
| Falling "piew-piew" | A **vehicle**: motorcycle, car, bicycle, bus or truck |
| Low buzzy "tud-tud" | An **object**: chair, table, bench, plant pot, etc. |
| Beeps get faster → nearly continuous | Closer (about 0.9 s apart at the edge of range, nearly continuous under 1 m) |
| Left ear / center / right ear | The obstacle's direction. Left is also slightly lower in pitch and right slightly higher, so a single speaker still works |

**Spoken warnings, from most to least urgent:**
1. **Rushing in:** "ระวัง มอเตอร์ไซค์ พุ่งเข้ามา ข้างหน้า 7 เมตร" ("watch out, motorcycle rushing in, ahead, 7 m"). Something is closing fast and will reach your path within about 4 s.
2. **Cutting across:** "รถยนต์ กำลังตัดผ่าน จากซ้าย 5 เมตร" ("car crossing from the left, 5 m"). Something is moving sideways into your walking path.
3. **Get-ready zone (2 m):** "เก้าอี้ ขวางทาง 2 เมตร" ("chair blocking the path, 2 m"). The app names whatever is standing in your path.

Tap the middle of the screen to hear a spoken summary of all 3 directions.

## How it works
1. **Detection:** COCO-SSD (TensorFlow.js) runs fully in the browser, using WebGPU when available and falling back to WebGL. No video leaves the phone.
2. **Far range (7–10 m):** every 3rd frame the app runs a second pass on the **zoomed-in center** of the image, where distant things in your path appear. This makes far objects twice as large for the model.
3. **Distance:** a pinhole-camera estimate from each object's known real-world size. It uses the closer of the height-based and width-based estimates, so mistakes err on the safe side. It can be calibrated with a person standing 2 m away.
4. **Position in meters:** the app converts each object's position into a sideways offset (X) in meters. It compares X with a 1.2 m-wide walking corridor to decide whether something is **in your path**.
5. **Motion:** objects are tracked between frames. Least-squares slopes over the last 1.2 s give closing speed and sideways speed. These are used to predict whether an object will enter the corridor. Warnings must hold for 2 consecutive frames to filter out noise. People walking toward you at normal pace don't trigger a "rushing in" warning, only joggers or faster.
6. **Sound:** Web Audio API, with a separate beep rhythm and stereo pan for each sector.

Tested in simulation (50 noisy runs each): a motorcycle at 6 m/s is warned at about 7 m, a car crossing is warned at about 5–6 m, a chair in the path is announced at 2 m, and a parked car beside the path or a normal walking person does **not** trigger "rushing in".

## How to run
**Phone:** open the live demo link, tap the big yellow button, and allow camera access. Earphones are recommended (open-ear or one earbud, so you can still hear traffic). The small text in the top right corner shows the frame rate (fps) and the GPU mode.

**Computer (testing):** `python -m http.server 8000`, then open http://localhost:8000. The camera only works over HTTPS or on localhost.

**Settings (⚙, for a sighted helper):** range (3–10 m), get-ready zone, volume, the speed that counts as "rushing in", model choice (fast or accurate), zoom pass, WebGPU, path-clear sound, vibration, and distance calibration.

## Limitations
- It only detects about 20 known object types. It **cannot see** walls, poles, holes, steps or curbs.
- Distance is estimated from typical object sizes. Expect about ±30% error, and more beyond 7 m. Objects at 10 m are small in the image, so they may be missed, especially with the "fast" model.
- Body sway while walking adds noise to sideways motion, so holding the phone steady helps.
- On iPhone, turn off silent mode if there is no sound. iPhone does not support vibration.
- **Always use it with a white cane.**

## Tech
HTML/CSS/JS · TensorFlow.js (WebGPU/WebGL) · COCO-SSD · Web Audio API · Web Speech API (th-TH) · Vibration API · Screen Wake Lock API
