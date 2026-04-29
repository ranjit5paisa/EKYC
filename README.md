# EKYC Lead Generation Parameters

This document details all parameters passed to EKYC during the lead generation process on the 5paisa website.

---

## 1. UTM Parameters

Extracted from `__gtm_campaign_url` cookie in [`getUrlParamsDetails()`](../themes/custom/fivepaisa/js/kyc-oda-form.js:220):

| Parameter | Description | Data Type | Example Value | Required | Notes |
|-----------|-------------|-----------|---------------|----------|-------|
| `utm_source` | Traffic source identifier | String | `partnerpage_website`, `google`, `facebook` | No | Default for /partner page: `partnerpage_website` |
| `utm_medium` | Marketing medium | String | `partner_organic`, `cpc`, `email` | No | Default for /partner page: `partner_organic` |
| `utm_campaign` | Campaign name | String | `partner_organic`, `summer_sale` | No | Default for /partner page: `partner_organic` |
| `utm_term` | Paid search keywords | String | `demat+account` | No | Optional |
| `utm_content` | Ad content differentiator | String | `banner_v1` | No | Optional |

### Code Snippet (Lines 243-272):
```javascript
const campaignCookie = myCookies.__gtm_campaign_url;
if (campaignCookie) {
  const upurl = new URL(campaignCookie);
  [
    "utm_source",
    "utm_medium",
    "utm_campaign",
    "utm_term",
    "utm_content",
  ].forEach((param) => {
    let value = upurl.searchParams.get(param);
    // Default values for /partner page
    if (!value && pathName == '/partner') {
      switch (param) {
        case "utm_source": value = "partnerpage_website"; break;
        case "utm_medium": value = "partner_organic"; break;
        case "utm_campaign": value = "partner_organic"; break;
      }
    }
    if (value) urlParams.append(param, value);
  });
}
```

---

## 2. Tracking Parameters

| Parameter | Description | Source Location | Data Type | Example Value | Required | Notes |
|-----------|-------------|-----------------|-----------|---------------|----------|-------|
| `Source_Identifier_URL` | Full page URL where lead originated | `window.location.href` or URL query param | String (URL encoded) | `https://www.5paisa.com/open-demat-account` | Yes | UTM params appended to this URL |
| `GA_Client_ID` | Google Analytics Client ID | `ga.getAll()[0].get('clientId')` | String | `123456789.1234567890` | No | Base64 encoded when redirecting |
| `WebsiteFormType` | Form location on page | Form wrapper element or URL param | String | `hero_form_d`, `hero_form_m`, `footer_form` | Yes | `_d` = desktop, `_m` = mobile |
| `type` | Same as WebsiteFormType | Derived from WebsiteFormType | String | `hero_form_d`, `hero_form_m`, `footer_form` | Yes | Duplicate of WebsiteFormType |

### Code Snippet (Lines 279-304):
```javascript
let cUrl = "";
try {
  cUrl = getParamFromCurrentURL("Source_Identifier_URL");
  if (!cUrl) cUrl = window.location.href;
  else cUrl = atob(cUrl);
} catch (e) {
  cUrl = window.location.href;
}

// ... later in the function ...

params = `${params}Source_Identifier_URL=${encoded_curl}&rcode=${encoded_rcode}&leadSource=${leadSource}&type=${getWebsiteFormType()}&WebsiteFormType=${getWebsiteFormType()}&GA_Client_ID=${encoded_GA_Client_ID}`;
```

---

## 3. User Input Parameters

| Parameter | Description | Source Location | Data Type | Example Value | Required | Notes |
|-----------|-------------|-----------------|-----------|---------------|----------|-------|
| `mobile` | User's mobile number | Form input `name='mobilenumber'` | String (10 digits) | `9876543210` | Yes | Base64 encoded when redirecting to OTP page |
| `otp` | One-time password | Form input `name='otp'` | String (6 digits) | `123456` | Yes | Only during OTP verification |
| `client_first_name` | User's first name | Form input (partner page only) | String | `paisa` | Yes | Default: `'paisa'` |
| `selected_account` | Account type selection | Form select element | String | `SE` (default) | No | Account type selected by user |

---

## 4. Referral Parameters

| Parameter | Description | Source Location | Data Type | Example Value | Required | Notes |
|-----------|-------------|-----------------|-----------|---------------|----------|-------|
| `rcode` | Referral/promo code (URL format) | URL query param `?rcode=` | String | `ABC123` | No | Converted to `referralcode` in backend |
| `referralcode` | Referral/promo code (standard format) | URL query param or user input | String | `ABC123` | No | Used for partner referrals |

### Code Snippet (Lines 144-157):
```javascript
let rcode = getDecodedParam("rcode") || getDecodedParam("referralcode");
if (checkdomStatus("promo")) {
  if (flags.isSamePageOtp && rcode) {
    rcode = atob(rcode);
  }
  if (rcode) {
    // TFS 166004 Hide brokerage section when ?ReferralCode in URL
    document.querySelector('.odabase-block__commission').classList.add('hide');
    flags.isReferralCodeFromURL = true;
    await verifyPromo(rcode);
    lockReferralCode();
  }
}
```

---

## 5. Lead Source Parameters

| Parameter | Description | Source Location | Data Type | Example Value | Required | Notes |
|-----------|-------------|-----------------|-----------|---------------|----------|-------|
| `leadSource` | Lead origin identifier | Derived from page path | String | `Website`, `Authorised_Partner` | Yes | `Authorised_Partner` for /partner page |

### Code Snippet (Line 303):
```javascript
let leadSource = window.location.pathname == '/partner' ? 'Authorised_Partner' : 'Website';
```

---

## 6. Partner Screening Parameters

Only applicable for leads from `/partner` page. Added in [`verifyOtpAndGenerateLead()`](../themes/custom/fivepaisa/js/kyc-oda-form.js:817):

| Parameter | Description | Source Location | Data Type | Example Value | Required | Notes |
|-----------|-------------|-----------------|-----------|---------------|----------|-------|
| `Partner_Score` | Partner screening score | Partner screening questionnaire | Number | `85` | No | Only for /partner page leads |
| `Partner_Status` | Partner approval status | Partner screening questionnaire | String | `Approved`, `Rejected` | No | Only for /partner page leads |
| `Partner_Questions` | JSON of screening answers | Partner screening questionnaire | JSON String | `{"q1": "a1", "q2": "a2"}` | No | Only for /partner page leads |

### Code Snippet (Lines 823-831):
```javascript
const screeningData = typeof window.getPartnerScreeningData === 'function' ? window.getPartnerScreeningData() : null;

if (screeningData) {
  const partnerParams = new URLSearchParams();
  partnerParams.append('Partner_Score', screeningData.Partner_Score);
  partnerParams.append('Partner_Status', screeningData.Partner_Status);
  partnerParams.append('Partner_Questions', screeningData.Partner_Questions);
  URLParams += '&' + partnerParams.toString();
}
```

---

## 7. System Generated Parameters

| Parameter | Description | Source Location | Data Type | Example Value | Required | Notes |
|-----------|-------------|-----------------|-----------|---------------|----------|-------|
| `ipAddress` | User's IP address | `$_SERVER['REMOTE_ADDR']` | String | `192.168.1.1` | Yes | Captured server-side |
| `checksum` | API authentication hash | Generated via MD5 hashing | String (16 char) | `A1B2C3D4E5F6G7H8` | Yes | Different algorithms for different APIs |
| `appSource` | Application source identifier | Hardcoded | String | `WEB` | Yes | Always `'WEB'` for website |

---

## 8. Backend API Parameters

Parameters sent to KYC API (`kycapi.5paisa.com/user/register`) in [`generateLead()`](../modules/custom/five_paisa/src/Controller/KycOdaFormApis.php:661):

| Parameter | Description | Source Location | Data Type | Example Value | Required | Notes |
|-----------|-------------|-----------------|-----------|---------------|----------|-------|
| `clientFirstName` | Client's first name | Form input or default | String | `paisa` | Yes | Sent to KYC API |
| `clientLastName` | Client's last name | Not collected | String | (empty) | No | Not currently collected |
| `product` | Product type | Hardcoded | String | `Equity` | Yes | Always `'Equity'` |
| `leadId` | Lead identifier | Empty on creation | String | (empty) | No | Generated by KYC API |
| `preSelectedPlan` | Pre-selected plan | Hardcoded | String | `Test` | Yes | Always `'Test'` |
| `email` | User's email | Not collected on initial form | String | (empty) | No | Collected later in KYC journey |
| `leadCampaign` | Campaign identifier | Empty | String | (empty) | No | Not currently used |
| `smsOTP` | Verified OTP | User input | String (6 digits) | `123456` | Yes | During lead generation |
| `userId` | User identifier (mobile) | Same as mobile | String (10 digits) | `9876543210` | Yes | Same as mobile number |
| `urlParams` | All URL parameters combined | Built from all above params | String (URL encoded) | `Source_Identifier_URL=...&rcode=...&leadSource=...` | Yes | Contains all tracking info |
| `emailVerified` | Email verification status | Empty | String | (empty) | No | Not used on initial form |
| `mobileVerified` | Mobile verification status | Empty | String | (empty) | No | Not used on initial form |
| `skipOtpSource` | Skip OTP source | Empty | String | (empty) | No | Not used |
| `emailConsent` | Email consent flag | Form input | String | `SE` | No | Same as selected_account |
| `mobileConsent` | Mobile consent flag | Form input | String | `SE` | No | Same as selected_account |
| `subSource` | Sub-source identifier | Derived from URL path | String | `All`, (empty for landing pages) | Yes | Empty for `/landing/*` pages |

### Complete Backend Payload (Lines 682-705 in KycOdaFormApis.php):
```php
$postData = '{
  "appSource" : "WEB",
  "mobile" : "' . $mobile . '",
  "clientFirstName" : "' . $ClientFirstName . '",
  "clientLastName" : "' . $ClientLastName . '",
  "product" : "Equity",
  "leadId" : "",
  "leadSource" : "Website",
  "preSelectedPlan" : "Test",
  "email" : "",
  "leadCampaign" : "",
  "smsOTP" : "' . $otp . '",
  "userId" : "' . $mobile . '",
  "urlParams" : "' . $URLParams . '",
  "referralCode" : "' . $promocode . '",
  "ipAddress" : "' . $ipaddress . '",
  "checksum" : "' . $checksum . '",
  "emailVerified" : "",
  "mobileVerified" : "",
  "skipOtpSource" : "",
  "emailConsent" : "' . $em_consent . '",
  "mobileConsent" : "' . $em_consent . '",
  "subSource" : "' . $subSource . '"
}';
```
