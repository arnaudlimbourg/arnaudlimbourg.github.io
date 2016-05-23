---
title: "Interacting with mailchimp with django"
---

I recently needed to interact with Mailchimp from a Django application.

After reading forums posts, looking at different options I ended up choosing the official Python package from Mailchimp. It lives on PyPi https://pypi.python.org/pypi/mailchimp. This package seemed to be the best option as it is up-to-date whereas alternatives like django-mailchimp-1.3 do not seem to be. I was also not sure which ones work with Mailchimp API v2.

One important thing to note is mailchimp python package code is generated and is not the most readable. All parameters are documented so with a bit of reading you can get most of what you need. Even though it is not as straightforward as it could.

A simple example to add a user to a mailing-list with additionnal merge vars looks like the following. Hopefully it will help and put you on the right track.

```python
from django.conf import settings


def add_user_to_list(user):
    api_key = settings.MAILCHIMP_API_KEY

    try:
        m = mailchimp.Mailchimp(api_key)
    except ConnectionError:
        print "Cannot connect to Maichimp"
        return False

    try:
        m.helper.ping()
    except mailchimp.Error:
        print "Invalid API key"
        return False
    except ConnectionError:
        print "Cannot connect to Maichimp"
        return False

    merge_vars = {
        'FNAME': user.first_name
    }

    try:
        m.lists.subscribe(
            settings.MAILCHIMP_LIST_ID,
            {'email': user.email},
            merge_vars=merge_vars,
            double_optin=False
        )
    except (ListDoesNotExistError, EmailNotExistsError, ListAlreadySubscribedError) as e:
        print (
            "Cannot add user {0} to the list, an exception occured {1}".format(
                user, type(e).__name__
            )
        )
```

When adding a user you can pass the optionnal parameter double_optin, it avoids sending a confirmation email to the person asking for consent. As Mailchimp indicates you do not want to abuse this, so make sure you use it responsibly.
