# Partner onboarding — source requirements

Defines three flows for an experience-platform
business onboarding tour/activity partners:

## 1. Single-partner analysis (in scope)

Volume: ~500/year. Input: a partner's website URL.

Summarize the provider and its portfolio; identify and structure its
sellable products (core / variant / add-on); research public reviews
(Google, Tripadvisor, optionally Trustpilot); score the partner
against a predefined criteria catalog; output a partnership
recommendation with priority (top candidate / to monitor / do not
pursue) plus optional manual-check hints (liability, insurance,
capacity).

Worked example in the source doc: QUADFARM guided quad tours.

## 2. Search & prioritization of potential partners (in scope)

Volume: ~2,000/year.

Accept freely definable, reusable search criteria (region, experience
type, target audience, price segment, minimum rating, site language).
Automate discovery via search engines, directories, review portals,
and social channels — each candidate found is represented by its
website URL. Score candidates with the same standardized model as flow
1; produce a prioritized, justified list; then create leads.

## 3. Self-onboarding of individual experiences (deferred, out of scope)

Volume: ~1,000/year, roughly 100 fields each.

A provider submits the URL of a single experience detail page; the
system extracts and maps its content (title, descriptions, included
services, location, duration, price, image URLs, terms) into the
onboarding form, using AI/rule logic for picklist fields with manual
override. The provider previews and corrects; on confirmation, data is
stored and optionally routed to internal review/approval.