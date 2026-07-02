# Namoza Developer Assignment

**Candidate:** Mohd Kamil
**Role:** Developer - Position 1 (Client Web + Martech)
**Loom Walkthrough:** [Insert your Loom video link here]

---

## Task 1: GTM Event Schema

### 1. Tagging Architecture Table

| Event Name | Trigger Type | Key Parameters | GA4 Destination |
| :--- | :--- | :--- | :--- |
| `booking_step_complete` | Custom Event: `booking_step_complete` | `step_number`, `step_name`, `clinic_location`, `specialty` | Funnel Exploration / Remarketing |
| `click_call_now` | Click - Just Links (`tel:`) | `page_location`, `button_position`, `phone_number_dialed` | Events Report / Mid-Intent |
| `click_whatsapp` | Click - Just Links (`wa.me`) | `page_location`, `click_text`, `clinic_location` | Events Report / Mid-Intent |
| `generate_lead_guide` | Custom Event: `guide_form_submitted` | `lead_name`, `lead_phone`, `guide_name` | Conversions / Nurture Audience |
| `view_clinic_location` | Page View (URL contains `/clinic/`) | `clinic_name`, `city`, `page_referrer` | Pages & Screens / Local Retargeting |
| `scroll_blog` | Scroll Depth (25, 50, 75, 90%) | `percent_scrolled`, `article_title`, `blog_category` | Engagement Report |

### 2. The 3-Step Booking Form Drop-off Logic
GTM cannot natively listen to multi-step form interactions reliably. The front-end developer must write custom `window.dataLayer.push()` events that fire only after each specific step is successfully validated and completed.

**Step 1: Location & Specialty Selected**
```json
{
  "event": "booking_step_complete",
  "step_number": 1,
  "step_name": "location_specialty_selected",
  "clinic_location": "Indiranagar",
  "specialty": "Knee Replacement"
}

Step 2: Patient Details Entered

{
  "event": "booking_step_complete",
  "step_number": 2,
  "step_name": "patient_details_entered",
  "preferred_date": "2026-07-15",
  "lead_type": "new_patient",
  "clinic_location": "Indiranagar"
}

Step 3: Booking Confirmed

{
  "event": "booking_step_complete",
  "step_number": 3,
  "step_name": "booking_confirmed",
  "booking_id": "ON-98765",
  "specialty": "Knee Replacement",
  "clinic_location": "Indiranagar"
}

Surfacing in GA4:
To visualize drop-off, I would create a Funnel Exploration report in GA4.

Step 1 Condition: Include event booking_step_complete where step_number exactly matches 1.

Step 2 Condition: Include event booking_step_complete where step_number exactly matches 2.

Step 3 Condition: Include event booking_step_complete where step_number exactly matches 3.
