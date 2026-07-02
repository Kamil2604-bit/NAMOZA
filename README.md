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

**Task 3: Integration Design

**Integration Architecture End-to-End**
To ensure low latency and high reliability, I would split the architecture into client-side tracking and server-side automation using **Make (formerly Integromat)** as the central orchestrator.

1. **Google Ads Conversion (Client-Side):** The conversion event must fire instantly. This is handled on the front end by the GTM `dataLayer.push` built in Task 2. GTM intercepts this payload and fires the Google Ads conversion tag directly in the user's browser, allowing the campaign to optimise immediately.
2. **CRM & WhatsApp Flow (Server-Side via Make):** Upon form submission, the vanilla JS executes an asynchronous `fetch()` POST request sending the Name, Phone, and Clinic Preference to a Make Webhook. Make is chosen over Zapier because its visual router and error-handler modules offer superior control for SLA monitoring and complex API lookups.

**The Biggest Failure Point & Fallback Strategy**
The most critical failure point lies within HubSpot’s native data model. By default, HubSpot deduplicates contacts using email addresses. Because this healthcare form intentionally omits the email field to reduce user friction, sending raw data to the standard HubSpot Forms API will result in massive duplicate records (e.g., if a user submits twice, or books for a family member).

To build a fallback for this, the Make scenario will not use a simple "Create Contact" module. Instead, it will first execute a "Search Contacts" API call using the submitted Phone Number. 
* **If a match is found:** Make updates the existing record with the new 'Clinic Preference' and resets Lead Status to 'New Enquiry'.
* **If no match is found:** Make creates a new contact with Name, Phone, Clinic Preference, Source = 'Google Ads - Consultation Landing Page', and Lead Status = 'New Enquiry'.

**WhatsApp 2-Minute SLA & Monitoring**
Once the HubSpot routing completes, Make triggers an HTTP POST request to the Karix WhatsApp Business API. 

The 2-minute SLA is most likely to break due to Karix API rate-limiting, temporary Karix server outages, or Make webhook queue delays during high-traffic spikes. 

To monitor and protect this SLA, I would attach an Error Handler module to the Karix HTTP request in Make. If the request takes longer than 60 seconds to resolve or returns a 5xx status code, the Error Handler instantly routes a fallback alert to a developer Slack channel. It will simultaneously push the failed payload into a Google Sheet queue for manual recovery once Karix resolves its downtime.
