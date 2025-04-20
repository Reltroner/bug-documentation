# 📊 Exclude Internal Traffic from GA4 using GTM and `traffic_type=internal`

This guide explains how to exclude your own visits from being tracked in Google Analytics 4 (GA4), using a URL-based filter through Google Tag Manager (GTM). Ideal for solo developers or creators with dynamic IPs (e.g. working from cafes, hotspots, or changing networks).

---

## ✅ Purpose

To ensure that your GA4 reports **do not count internal or self-traffic**, especially useful when building and testing your own website (e.g. `reltroner.com`), by using:

```
https://reltroner.com/?traffic_type=internal
```

---

## 🛠️ What You'll Build

- A custom **GTM Trigger** that detects `traffic_type=internal` in the URL
- A GA4 **Custom Event Tag** that sends the `traffic_type` parameter
- A **GA4 Data Filter** that excludes this internal traffic permanently

---

## 🔧 Step-by-Step Setup

### 1. Create a URL Variable in GTM

- Go to **Variables** → `New`
- Choose Variable Type: `URL`
- Component Type: `Query`
- Query Key: `traffic_type`
- Save as:  
  ```
  URL – traffic_type
  ```

---

### 2. Create Trigger – Internal Traffic

- Go to **Triggers** → `New`
- Trigger Type: `Page View`
- Trigger when:  
  ```
  URL – traffic_type equals internal
  ```
- Name it:  
  ```
  Trigger – Internal Traffic
  ```

---

### 3. Create GA4 Event Tag

- Go to **Tags** → `New`
- Tag Type: `Google Analytics: GA4 Event`
- Measurement ID: (your GA4 ID, e.g. `G-XXXXXXXXXX`)
- Event Name: `page_view`

#### Under Event Parameters:
| Name         | Value    |
|--------------|----------|
| traffic_type | internal |

- Trigger: use `Trigger – Internal Traffic`
- Save

---

### 4. Connect GTM to your site and Test

- Use **Tag Assistant** to preview:  
  [https://tagassistant.google.com](https://tagassistant.google.com)
- Visit your site with:  
  ```
  https://reltroner.com/?traffic_type=internal
  ```
- Confirm `GA4 Event` tag fires with parameter `traffic_type: internal`

---

### 5. Enable GA4 Data Filter

- Go to GA4 Admin → **Data Settings** → **Data Filters**
- Click **Create Filter**
- Type: `Internal Traffic`
- Filter name: `Exclude My Personal IP`
- Filter operation: `Exclude`

#### Filter parameter:
| Parameter name | Parameter value |
|----------------|------------------|
| traffic_type   | internal         |

- Filter State: ✅ `Active`
- Save

---

## 🧪 How to Use

To mark a session as internal (excluded):

```
https://reltroner.com/?traffic_type=internal
```

You can also **bookmark this version of your site** when working on development or content creation.

---

## 💬 Notes

- No need to rely on fixed IPs.
- Works on any browser/device.
- If you work with a team, ask them to use `?traffic_type=internal` or set up additional cookie logic.

---

## 🛡️ Why This Matters

> _“You don’t want your own journey to pollute your signal to the world.”_  
> Use this to ensure your insights are **pure, accurate, and unbiased**.

Let Astralis light the unknown.  
— Maintained by [Rei Reltroner](https://reltroner.com)
