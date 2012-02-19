---
layout: post
title: Localization with locale aware routes in python pyramid
---

Localization is constantly losing im importance it appears to me. Of course English nis becoming more and more common, but then I still work in good ole Germany and here my Granny cannot speak a word of that language. If targetting a tech savvy elite is your goal, sure, English is the new Black. But when you are targetting everyone, there is still some way to go.

Generally some people have the money to buy country specific domains and then distinguishing locales becomes as excercise in figuring out which domains the site is being called on. Some just run languages on sub domains, same difference. For smaller sites however rewriting URL paths into locale aware URLs is the way to go (aka rewriting /content/aboutus -> /en/content/aboutus). Now even this can be quickly done with some rewrite rules and set_header magic. However this puts some burden onto the dployment process when available locales change. So, the following quickly outlines how you could rewrite routes centrally with python pyramid.

[This article](http://my.opera.com/WebApplications/blog/2008/03/17/search-engine-friendly) from the opera blog inspired much of the work back when I used pylons, most of their words apply today, even though actual code is quite different.

Mostly you need to be aware of 2 parts. 1) reading locale from incoming requests, and if no information is available, negotiate a locale and set it for the user as well as redirect him/her to the correct locale aware destination destination (routing). 2) generating locale aware links through out your application.

== 1)Routing
Pyramid even has a [default locale negotiator](http://docs.pylonsproject.org/projects/pyramid/en/latest/api/i18n.html#pyramid.i18n.default_locale_negotiator), with this one could already generate paths that include the ugly _LOCALE_ request parameter, it would even be easy to just implement your own verion of it, as it is just:

<pre class="prettyprint linenums language-python">
def default_locale_negotiator(request):
    name = '_LOCALE_'
    locale_name = getattr(request, name, None)
    if locale_name is None:
        locale_name = request.params.get(name)
        if locale_name is None:
            locale_name = request.cookies.get(name)
    return locale_name
</pre>

However I've seen more than once this negotiator being invoked too late for some redirects, aka at a time when locale had not been negotiated. So instead you could just hook it into Pyramid's event system, e.g. [event.NewRequest](http://docs.pylonsproject.org/projects/pyramid/en/latest/api/events.html#pyramid.events.NewRequest). However, this is pyramid, you can do pretty much anything, so now hook straight into request object creation?

A short sample config:

<pre class="prettyprint linenums language-python">
    config = Configurator(settings=settings
              , request_factory = ExtendedRequest
              , renderer_globals_factory = add_renderer_globals
              , session_factory = session_factory_from_settings(settings)
              )
</pre>

Here you see an ExtendedRequest, that will be used for any incoming request (i.e. also for static routes as long as they route through your python layer, but really, who is doing that?). Now you can simply read the locale in a dynamic property from the URL (be aware that matched_route is not available at object creation nor at events.NewRequest, route matching happens afterwards).

With this cou create a dynamic property on 

<pre class="prettyprint linenums language-python">
class ExtendedRequest(Request):
  ...
  def get_LOCALE_(self):
    locale = getattr(self, '__LOCALE__', None)
    if(locale): return locale
    elif('_LOCALE_' in self.cookies):
      self.__LOCALE__ = self.cookies['_LOCALE_']
      return self.__LOCALE__
    else:
      locale = self.registry.settings['pseudo_locale_negotiator'](self)
      log.info("SETTING UP LOCALE %s", locale)
      self.__LOCALE__ = locale
    return locale
  def set_LOCALE_(self, locale):
    self.__LOCALE__ = locale
    self.add_response_callback(setLangCookie)
  _LOCALE_ = property(get_LOCALE_, set_LOCALE_)
</pre>

Where pseudo_locale_negotiator is an instane of a negotiator for only a new user.

<pre class="prettyprint linenums language-python">
from babel import negotiate_locale

class PseudoLocaleNegotiator(object):
  def __init__(self, available_locales, default_locale_name):
    self.available_locales = available_locales
    self.default_locale_name = default_locale_name
  def __call__(self, request):
    accept_langs = request.accept_language
    def normalize_locale(loc):
      return unicode(loc).replace('-', '_')
    langs = map(normalize_locale, accept_langs)
    locale = negotiate_locale(langs, self.available_locales, sep="_") or self.default_locale_name
    request.add_response_callback(setLangCookie)
    return locale
</pre>

This will mark any user with a locale, one which WebOb and babel together negotiate.

However there might be links out there with locale-unaware URLs. If you want to catch them as well, youll need a little NotFound context view. This is easy enough as pyramid provides this little helper with the very Java-like name [AppendSlashNotFoundViewFactory](http://docs.pylonsproject.org/projects/pyramid/en/latest/api/view.html?highlight=appendslash#pyramid.view.AppendSlashNotFoundViewFactory), which we can just adapt to:

<pre class="prettyprint linenums language-python">
class TryLanguageNotFoundViewFactory(object):
    def __init__(self, notfound_view=None):
        if notfound_view is None:
            notfound_view = default_exceptionresponse_view
        self.notfound_view = notfound_view

    def __call__(self, context, request):
        path = request.path
        registry = request.registry
        mapper = registry.queryUtility(IRoutesMapper)
        if mapper is not None and not path.startswith('/{}'.format(request._LOCALE_)):
            localepath = "/{}{}".format(request._LOCALE_, path).rstrip("/")
            for route in mapper.get_routes():
                if route.match(localepath) is not None:
                    qs = request.query_string
                    if qs:
                        localepath += '?' + qs
                    return HTTPMovedPermanently(location=localepath)
        return self.notfound_view(context, request)
</pre>

And register this view via:

<pre class="prettyprint linenums language-python">
  def notfound_view(context, request):
    return HTTPNotFound('It aint there, please stop trying now!')
  custom_locale = TryLanguageNotFoundViewFactory(notfound_view)
  config.add_view(custom_locale, context=HTTPNotFound)
</pre>

This will get any user back on track to their locale aware destintation. Please not this little part <code>path.startswith('/{}'.format(request._LOCALE_))</code>, which prevents this NotFoundView from repeatedly adding locale information in case of an actual not found page.

== 2) URL Generation
This specific implementation does not prevent you from actually adding locale to any generated URL throughout your application. A simple wrapper around <code>request.route_path</code> can help.

<pre class="prettyprint linenums language-python">
class ExtendedRequest(Request):
  ...
  def locale_path(self, *args, **kwargs):
    return self.route_path(*args, lang=self._LOCALE_, **kwargs)
</pre>

How it would be even better if we could in the ExtendedRequest constructor extract the locale information, generate routes completely inaware of any locale information and provide a hook into URL Generation, which would add it back in. Especially as routing cuold then be completely unaware of any locale information. Alas, that is for another day.

---

Today however this is the small part of I18N problems. The bigger one is keeping the localizations up to date with as little man power as possible. Especially with today's rich javascript clients this is becoming more and more of a challenge. Ever tried to explain to a user, why immediate validation (maybe jquery.validate) does use the correct language, but when you submit the form, it actually supplies a different message from the python backend? Now you have to hook your localization into [jQuery.validate({messages:{}})](http://docs.jquery.com/Plugins/Validation/validate#options). Sure sure. do that, while Ill use my same validation code with validate and node, identical validation and messages for free.

That's how I like it (some other day).
