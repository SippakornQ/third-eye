# ตาที่สาม (Third Eye) — A Phone Sonar for Blind and Low-Vision Walkers

A web app that turns a phone camera into a **3-direction sonar with spoken warnings in Thai**. Hang the phone around your neck with the back camera facing forward. It beeps faster as obstacles get closer, and **people, vehicles and objects (including electric poles)** each have their own sound. It speaks **short, clear warnings with what it is, which direction, and how far**: a vehicle coming at you from **12 m**, an obstacle in your path from **6 m**, and people or things within **2 m** in front of you or to your side (the warning for people can be switched on or off). Things it cannot name are called **"วัตถุ"** ("object").

**Live demo:** https://sippakornq.github.io/third-eye/ (open on a phone)

## The problem
Thai sidewalks are crowded with electric poles, parked and moving motorcycles, vendor stalls, chairs and people. A white cane only detects what it touches, about one step ahead and at ground level. It cannot tell a blind walker that a motorcycle is coming at them, that a car is about to cut across, or what is blocking the way a few meters ahead. Without help, blind and low-vision people have to walk slowly, bump into things, or depend on someone to guide them.

## Who it is for
Blind and low-vision people walking on sidewalks, around campus, or around their dorm. It is a helper to use **together with a white cane**, not a replacement for one.

## Zones and warnings

| Zone | What happens | Example (spoken) |
|---|---|---|
| **Vehicles, up to 12 m** | A vehicle rushing toward you or cutting across your path | "ระวังรถ ซ้าย 10 เมตร" ("watch out, car, left, 10 m") · "รถตัดหน้า จากขวา 9 เมตร" ("car cutting across from the right, 9 m") |
| **Normal zone, 4–6 m and beyond** | An obstacle standing in your walking path | "เสา ข้างหน้า 6 เมตร" ("pole ahead, 6 m") · "วัตถุ ข้างหน้า 5 เมตร" ("object ahead, 5 m") |
| **Near zone, 0–2 m** | A person or thing in front of you or up to about 1 m to the side. The warning for people can be switched on/off | "คน ข้างหน้า ครึ่งเมตร" ("person ahead, half a meter") · "คน ซ้าย 1 เมตร" ("person left, 1 m") · "เก้าอี้ ขวา 2 เมตร" ("chair right, 2 m") |

Every warning follows the pattern **what · direction · distance**. Only the most important one is spoken at a time, and each one has a cooldown so the app doesn't keep talking.

**Sonar sounds:** a high "tid" means a person, a falling "piew" means a vehicle, and a low "tud" means an object or pole. The beeps get faster as the obstacle gets closer. Left, center and right come through as stereo, with a slight pitch difference so a single speaker still works. A soft slow "pok" means the path is clear.

## How to use
1. Open the link on your phone and tap the big yellow **เริ่ม** (Start) button. Allow camera access.
2. Hang the phone around your neck with the back camera facing forward.
3. While it runs, the screen has only two big buttons: **หยุด** (Stop) and **เตือนคนใกล้ เปิด/ปิด** (near-person warning on/off). Tap anywhere else on the screen to hear a short summary of all 3 directions.
4. ⚙ **Settings:** the main options are simple switches. The advanced section, for a sighted helper, has the distances, the model choice and distance calibration.

## How it works
1. **Two AI models running in the browser** (TensorFlow.js, WebGPU when available, otherwise WebGL). No video leaves the phone.
   - **COCO-SSD** detects people, vehicles, animals and furniture quickly, every frame.
   - **DeepLab v3 (Cityscapes)**, a street-scene segmentation model, finds **electric poles**, which COCO-SSD cannot detect. It also finds **anything else standing in the walking path** that is not floor, sky, a person or a vehicle. These are named "ต้นไม้" (tree) or "ป้าย" (sign) when the model recognizes them, and "วัตถุ" (object) otherwise. It runs about once per second.
   - Distance to these unknown things doesn't need their size. It comes from **where they touch the ground**: distance = phone height × focal length ÷ pixels below the horizon. The phone's **tilt sensor** gives the horizon position. The phone height (default 1.3 m for a neck-hung phone) can be set in Settings.
2. **Far range:** every other frame, the app runs a second pass on the zoomed-in center of the image, so vehicles 10–12 m away are big enough for the model to detect.
3. **Distance** is estimated from each object's known real size: a person is about 1.65 m tall, a car about 1.75 m wide, a pole about 0.3 m wide. It can be calibrated with a person standing 2 m away. Between pole-model updates, the pole's distance keeps counting down at your walking speed.
4. **Telling things apart better:**
   - Each tracked object keeps the label it has been given most often recently, so a brief misdetection (a person seen as a chair for one frame) doesn't change what is announced.
   - A person sitting on a motorcycle is counted as part of the motorcycle.
   - Objects need higher confidence than people and vehicles before they count.
   - An object must be seen several times before anything is spoken.
5. **In the path or not:** each object's sideways position is converted to meters and compared with a 1.2 m-wide walking corridor. Closing speed and sideways speed (least-squares over the last 1.2 s) are used to predict "rushing in" and "cutting across".

**Simulation results** (40 noisy runs each):
- A car approaching at 8 m/s is warned at about 11–12 m, and a motorcycle at 6 m/s at about 12 m.
- A car crossing the path is warned at about 9 m.
- A pole in the path is announced at 6 m and again under 2 m.
- A person standing 0.5 m ahead is announced every 4 s, or never when switched off.
- A parked car beside the path is never announced.

## How to run
- **Phone:** open the live demo link.
- **Computer:** run `python -m http.server 8000`, then open http://localhost:8000. The camera only works over HTTPS or on localhost.
- **Deploy:** go to the repo's Settings → Pages → Source: `main` branch, `/ (root)`.

The small text in the top right corner shows the frame rate (fps), the GPU mode, the model (accurate or fast), and whether pole detection is working (เสา ✓ / ✗). If the accurate model is too slow on your phone, the app switches to the fast model automatically.

## Limitations
- Pole detection depends on the street-scene model. Thin, far-away, or unusual poles can be missed. Pole detection updates about once per second, not every frame.
- It does not detect holes, steps or curbs.
- Distance to unknown objects depends on the phone's tilt and height. A 5° tilt error can change the distance by about 50%. The tilt sensor corrects for this. If your phone has none, set the height in Settings and keep the phone upright.
- Distance is an estimate, about ±30%, and less accurate beyond 8 m.
- On iPhone, turn off silent mode if there is no sound. iPhone does not support vibration.
- **Always use it with a white cane.**

## Tech
HTML/CSS/JS · TensorFlow.js (WebGPU/WebGL) · COCO-SSD · DeepLab v3 Cityscapes · Web Audio API · Web Speech API (th-TH) · Vibration API · Screen Wake Lock API
