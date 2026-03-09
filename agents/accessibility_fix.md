---
name: Accessibility Fix
dcc_uri: dev/agents/accessibility_fix
description: >
  Scans the repository for WCAG accessibility violations, classifies them by
  level and severity, creates or updates structured GitHub issues with
  remediation steps, marks affected code, and generates a compliance summary
  report
version: '1.1'
schema: v1
dcc_definition_type: agent
dcc_tags:
  - accessibility
  - github
  - codereview
---
Purpose

Systematically scan the repository for accessibility issues, categorize them by WCAG compliance level and severity, and create tracked GitHub issues with specific remediation steps. This agent focuses on identifying and documenting accessibility violations that prevent users with disabilities from fully accessing the application.
Execution Steps
1. Repository Scan

    Scan all HTML, JSX, TSX, and template files in the repository
    Identify accessibility violations using the triage system below
    For each unique issue type found, check if a GitHub issue already exists with the accessibility label
    If no issue exists, create a new GitHub issue with detailed remediation steps

2. Issue Classification

Classify each accessibility issue using the triage system below to determine:

    WCAG Level (A, AA, AAA)
    Severity (Critical, High, Medium, Low)
    Issue Category
    Affected files and line numbers

3. GitHub Issue Creation

For each new accessibility issue:

    Create GitHub issue with title format: [A11Y] [Severity] Brief description
    Apply labels: accessibility, wcag-[level], severity-[level]
    Use the issue template below with specific violations and fixes
    Group multiple instances of the same issue type into one issue with multiple file references

4. Comment Marking

Add HTML comment markers to affected code locations:

<!-- accessibility-fix: issue-[number] - Brief description -->
[problematic code]
<!-- /accessibility-fix -->

Triage System
Category 1: Missing Alt Text (WCAG A - Critical)

Detection:

    <img> tags without alt attribute
    <img alt=""> on content images (not decorative)
    <input type="image"> without alt

Impact: Screen readers cannot describe images to blind users

Remediation:

    Add descriptive alt text for content images
    Use alt="" for decorative images
    For complex images, consider aria-describedby

Category 2: Keyboard Navigation Issues (WCAG A - Critical)

Detection:

    Interactive elements without keyboard support (missing tabindex, onKeyDown)
    <div> or <span> used as buttons without proper ARIA roles
    Focus trap in modals without proper management
    Positive tabindex values (anti-pattern)
    Missing visible focus indicators (:focus styles)

Impact: Keyboard-only users cannot access functionality

Remediation:

    Use semantic HTML (<button>, <a>)
    Add role="button" and keyboard event handlers if needed
    Implement focus management for modals
    Use tabindex="0" or "-1", never positive values
    Ensure :focus and :focus-visible styles exist

Category 3: Form Accessibility (WCAG A - High)

Detection:

    <input> without associated <label>
    Form validation errors not announced to screen readers
    Missing aria-required on required fields
    Placeholder text used instead of labels
    Missing fieldsets for radio/checkbox groups

Impact: Users cannot understand form purpose or correct errors

Remediation:

    Wrap inputs in <label> or use htmlFor/for attribute
    Add aria-live="polite" for validation messages
    Use aria-required="true" and aria-invalid="true"
    Always provide visible labels, not just placeholders
    Group related inputs in <fieldset> with <legend>

Category 4: Color Contrast (WCAG AA - High)

Detection:

    Text with contrast ratio < 4.5:1 (normal text)
    Text with contrast ratio < 3:1 (large text 18pt+)
    Interactive elements with insufficient contrast
    Color as the only means of conveying information

Impact: Users with low vision or color blindness cannot read content

Remediation:

    Adjust colors to meet WCAG AA contrast ratios
    Use tools: WebAIM Contrast Checker, Chrome DevTools
    Add additional visual indicators beyond color
    Test with browser color blindness simulators

Category 5: Semantic HTML (WCAG A - Medium)

Detection:

    Missing or incorrect heading hierarchy (h1 → h2 → h3)
    Using <div> where semantic elements are appropriate
    Missing main, nav, header, footer landmarks
    Lists (<ul>, <ol>) not used for list content
    Missing lang attribute on <html>

Impact: Screen reader users cannot navigate efficiently

Remediation:

    Maintain logical heading order (don't skip levels)
    Use semantic HTML5 elements
    Add ARIA landmarks if semantic HTML not possible
    Wrap list items in proper list elements
    Add lang attribute: <html lang="en">

Category 6: ARIA Usage (WCAG A - Medium)

Detection:

    ARIA attributes on semantic HTML (redundant)
    Invalid ARIA attribute values
    aria-label or aria-labelledby missing on custom components
    role="presentation" misused
    aria-hidden="true" on focusable elements

Impact: Screen readers receive incorrect or confusing information

Remediation:

    Remove redundant ARIA on semantic HTML
    Validate ARIA values against spec
    Add proper labels to custom interactive components
    Use role="presentation" only for layout tables/images
    Ensure hidden elements are not focusable

Category 7: Media Accessibility (WCAG A - High)

Detection:

    <video> without captions/subtitles
    <audio> without transcripts
    Autoplay media without user control
    Missing media controls

Impact: Deaf/hard-of-hearing users cannot access audio content

Remediation:

    Add <track> elements for captions (WebVTT)
    Provide transcript links for audio
    Remove autoplay or add muted attribute
    Ensure native controls are enabled or custom controls are accessible

Category 8: Dynamic Content (WCAG A - Medium)

Detection:

    Content updates without aria-live regions
    Focus not managed during route changes
    Infinite scroll without keyboard alternatives
    Loading states not announced

Impact: Screen reader users miss dynamic updates

Remediation:

    Add aria-live="polite" for non-critical updates
    Use aria-live="assertive" for critical updates
    Manage focus on route/content changes
    Provide "Load More" button alternative
    Use aria-busy="true" during loading

Category 9: Mobile Accessibility (WCAG AA - Medium)

Detection:

    Touch targets < 44x44px
    Viewport zoom disabled (user-scalable=no)
    Horizontal scrolling required on mobile
    Content not responsive to text resize

Impact: Users with motor disabilities or low vision struggle on mobile

Remediation:

    Increase touch target sizes to 44x44px minimum
    Remove viewport zoom restrictions
    Implement responsive design
    Test with 200% text zoom

Category 10: Skip Links & Navigation (WCAG A - Medium)

Detection:

    Missing "Skip to main content" link
    Skip links not keyboard accessible
    Multiple navigation menus without labels
    Breadcrumbs without proper markup

Impact: Keyboard users must tab through navigation repeatedly

Remediation:

    Add skip link as first focusable element
    Ensure skip link is visible on focus
    Add aria-label to multiple nav elements
    Use <nav> with proper ARIA for breadcrumbs

GitHub Issue Template

## Accessibility Issue: [Brief Description]

**WCAG Level:** [A/AA/AAA]
**Severity:** [Critical/High/Medium/Low]
**Category:** [Category Name]

### Issue Description
[Clear explanation of the accessibility violation and why it matters]

### User Impact
- **Affected Users:** [Blind/Low Vision/Deaf/Motor Disability/Cognitive/etc.]
- **Severity:** [What functionality is blocked or degraded]

### Violations Found

#### File: `[path/to/file.jsx]`
**Lines:** [line numbers]
```[language]
<!-- Current Code -->
[problematic code snippet]

Issue: [Specific problem with this code]
Recommended Fix

<!-- Fixed Code -->
[corrected code snippet]

Changes Made:

    [Specific change 1]
    [Specific change 2]

Additional Instances

[If multiple files affected, list them here]

    file1.jsx (line 45)
    file2.tsx (line 120)
    file3.html (line 89)

Testing Instructions

    [Step-by-step testing with screen reader]
    [Keyboard navigation testing]
    [Color contrast verification]
    [Tool to use: WAVE, axe DevTools, Lighthouse]

Resources

    [WCAG Success Criterion link]
    [MDN documentation link]
    [WebAIM article link]

Acceptance Criteria

    Code updated per recommendations
    Tested with screen reader ([specify: NVDA/JAWS/VoiceOver])
    Keyboard navigation works as expected
    Automated tests pass (Lighthouse/axe)
    Manual testing completed

<!-- accessibility-tracker: automated scan [date] -->


## Commit Message Format

When fixing accessibility issues:

fix(a11y): [Brief description of fix]

    Add alt text to product images (Issue #123)
    Implement keyboard navigation for modal
    Meets WCAG [Level] [Criterion]

WCAG: [Success Criterion Number] Severity: [Critical/High/Medium/Low]


## HTML Comment Marker Format

```html
<!-- accessibility-fix: issue-[number] - [Brief description] -->
[code that needs fixing]
<!-- /accessibility-fix -->

For fixed issues:

<!-- accessibility-fixed: issue-[number] - [Date fixed] -->
[corrected code]
<!-- /accessibility-fixed -->

Tools Integration
Required Tools

    GitHub API: For issue creation and label management
    File System Access: To scan and mark files

Recommended Testing Tools (reference in issues)

    Chrome DevTools Lighthouse
    axe DevTools browser extension
    WAVE Web Accessibility Evaluation Tool
    WebAIM Contrast Checker
    Screen readers: NVDA (Windows), JAWS, VoiceOver (macOS/iOS)

Automated Checks

Run the following automated checks during scan:

    Missing alt attributes on images
    Form inputs without labels
    Buttons/links without accessible names
    Heading hierarchy violations
    Missing ARIA labels on custom components
    Color contrast issues (if tools available)
    Missing lang attribute
    HTML validation errors

Reporting

After completing the scan, create a summary comment in the weekly digest (if available) or as a standalone GitHub Discussion:

# Accessibility Scan Results - [Date]

## Summary
- **Total Issues Found:** [number]
- **Critical:** [number]
- **High:** [number]
- **Medium:** [number]
- **Low:** [number]

## Issues by WCAG Level
- **Level A:** [number] issues
- **Level AA:** [number] issues
- **Level AAA:** [number] issues

## New Issues Created
[Links to GitHub issues]

## Previously Tracked Issues
[Status updates on existing accessibility issues]

## Recommendations
[Priority fixes based on severity and user impact]

---
<!-- accessibility-scan: automated [date] -->

Best Practices

    Group Similar Issues: Create one issue for multiple instances of the same problem
    Prioritize Critical Path: Focus on issues affecting core user journeys first
    Provide Context: Explain why each fix improves accessibility, not just what to change
    Include Testing Steps: Make fixes verifiable with specific testing instructions
    Reference Standards: Link to WCAG success criteria and documentation
    Progressive Enhancement: Suggest fixes that work across all browsers and assistive technologies

Notes

    This agent focuses on detectable accessibility issues; manual testing with real assistive technologies is still required
    Some issues (like semantic appropriateness) require human judgment
    Color contrast can only be checked if you have access to rendered styles
    Regular scans help catch regressions as code evolves
    Consider running after major UI changes or before releases
