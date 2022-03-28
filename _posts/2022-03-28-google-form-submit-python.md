---
title: "Automating Google form submission with python"
last_modified_at: 2022-03-24T14:08:00
categories:
  - Tech
tags:
  - Automation
  - Python
  - Google
  - Form
---

# Submitting Google form manually is painful
## Yeah Google form is useful but could it be better?

# Submitting Google form by URL
## Structure of Google form URL

Let's say you have Google form URL `https://docs.google.com/forms/d/e/1FAIpQLSdB0bVk3HzN62pG6SwGFJfvKX3tiCmowQr8ccRYjdaPwcZPkQ/viewform`

That's a long URL. I'm going to call `https://docs.google.com/forms/d/e/1FAIpQLSdB0bVk3HzN62pG6SwGFJfvKX3tiCmowQr8ccRYjdaPwcZPkQ` as `{fURL}` from here.

The last `viewform` in `{fURL}/viewform` states that you are viewing the Google form as viewing a form. It's that simple. But we do not want the view. We want the submission.

Let's change a little in the URL as `{fURL}/formResponse`
This is where you want to add submission responses but how? Let me show you the final results first.

```
{fURL}/formResponse?entry.255216653=Jongkuk Lim&entry.1145887023=Option 1&entry.1333424194=9999
```

You may have noticed `entry.{NUMBER}` have been added to the URL. Each entry number represents each questions. But how do we know which entry number is which question?

{% include figure
    image_path="assets/images/2022-03-28-google-form-submit-python/01_find_entry.png"
    alt="Find question component"
    caption="You can find question component in Chrome's inspection"
%} 

{% include figure
    image_path="assets/images/2022-03-28-google-form-submit-python/02_find_entry.png"
    alt="Find question component"
    caption="And find the entry's number here"
%} 

You will see entry numbers but don't be confused with the title entry number. In this case, you want the second entry number which has been highlighted.
Once you acquire all entry numbers, you can write simple Python code to test.

# Python implementation

```python
import requests

fURL = "https://docs.google.com/forms/d/e/1FAIpQLSdB0bVk3HzN62pG6SwGFJfvKX3tiCmowQr8ccRYjdaPwcZPkQ"
URL = f"{fURL}/formResponse?entry.255216653=Jongkuk Lim&entry.1145887023=Option 1&entry.1333424194=9999"

response = requests.post(URL)
```

This will do the trick. But the problem is that this method does not support the forms where Google login is required.

Let's dig more 🤔

