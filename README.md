# ตาที่สาม (Third Eye) — A Phone Sonar for Blind and Low-Vision Walkers

A web app that turns a phone camera into a **3-direction sonar**. Hang the phone around your neck or hold it at chest height, and it beeps for obstacles on the **left, center and right, up to about 5 m away**. The closer an obstacle is, the faster it beeps. People and objects make different sounds, and if something is **coming at you fast**, such as a motorcycle on the sidewalk, it gives a **spoken warning in Thai**.

**Live demo:** `https://<your-username>.github.io/<repo-name>/` (open on a phone)

## The problem
Thai sidewalks are crowded with parked and moving motorcycles, vendor stalls, tables, chairs and people. A white cane only detects what it touches, about one step ahead and at ground level. It cannot tell a blind walker that a motorcycle is speeding toward them, or that a person is about to walk into them. Without help, blind and low-vision people have to walk slowly, bump into things, or depend on someone to guide them.

## Who it is for
Blind and low-vision people walking on sidewalks, around campus, or around their dorm. It is a helper to use **together with a white cane**, not a replacement for one.

## How it works

| What you hear | Meaning |
|---|---|
| Soft, slow "pok … pok …" | Path clear: the system is running and nothing is within range |
| High clear "tid-tid-tid" | A **person** |
| Low buzzy "tud-tud-tud" | An **object**, such as a motorcycle, car, chair or table |
| Beeps get faster → nearly continuous | The obstacle is getting closer (about 0.9 s apart at 5 m, nearly continuous under 1 m) |
| Sound in the left ear / center / right ear | The obstacle's direction. Left is also slightly lower in pitch and right slightly higher, so a single speaker still works |
| Voice: "ระวัง มอเตอร์ไซค์ พุ่งเข้ามา ด้านซ้าย 4 เมตร" | Something is approaching fast ("motorcycle rushing in, on the left, 4 meters") |
| Tap the middle of the screen | Hear a spoken summary of all 3 directions |

Under the hood:
1. **Detection:** COCO-SSD (TensorFlow.js) runs fully in the browser, so no video leaves the phone. It recognizes people, vehicles, animals and street furniture.
2. **Distance:** a pinhole-camera estimate from each object's known real-world size (for example, a person is about 1.65 m tall). When a box is cut off by the edge of the frame, the app uses the closer of the height-based and width-based estimates, so mistakes err on the safe side. It can be calibrated by having a person stand 2 m away.
3. **Direction:** the frame is split into 3 sectors. Wide objects count in every sector they cover.
4. **Speed:** objects are tracked between frames. Closing speed is a least-squares slope of distance over the last second. A voice warning fires when a moving object (person, vehicle or animal) closes faster than 2.2 m/s and would reach you within 3 s. A walking person approaching at about 1.3 m/s, or a parked car, does **not** trigger it.
5. **Sound:** Web Audio API, with a separate beep rhythm and stereo pan for each sector.

## How to run
**Phone:** open the GitHub Pages link, tap the big yellow button, and allow camera access. Earphones are recommended (open-ear or one earbud, so you can still hear traffic).

**Computer (testing):**
```bash
python -m http.server 8000     # then open http://localhost:8000
```
The camera only works over HTTPS or on `localhost`.

**Deploy:** go to the repo's Settings → Pages → Source: `main` branch, `/ (root)`.

**Settings (⚙, for a sighted helper):** range, volume, the speed that triggers a voice warning, model choice (fast or accurate), path-clear sound, vibration, and distance calibration.

## Limitations
- It only detects about 20 known object types. It **cannot see** walls, poles, holes, steps or curbs.
- Distance is an estimate from typical object sizes, so expect roughly ±30%.
- It cannot tell your own walking from an object moving. That is why voice warnings use a speed threshold above normal walking pace and only apply to things that can move.
- On iPhone, turn off silent mode if there is no sound. iPhone does not support vibration.
- **Always use it with a white cane.**

## Tech
HTML/CSS/JS · TensorFlow.js · COCO-SSD · Web Audio API · Web Speech API (th-TH) · Vibration API · Screen Wake Lock API
