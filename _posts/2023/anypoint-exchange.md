---
title: "Ideas to improve Anypoint Exchange user experience"
date: 2023-01-10
categories:
 - MuleSoft
---

Anypoint Exchange is a key component of MuleSoft's Anypoint Platform. In my opinion, it is _the_ differentiator to allow Anypoint Platform stand apart in the crowded market of API management and integration/ESB software vendors. Unfortunately MuleSoft hasn't paid the required attention to Anypoint Exchange since I first started using it in 2017. The newer search functionality introduced in 2020 was atrocious. Other than a few cosmetic changes around background color and support for tags, Anypoint Exchange became _significantly_ worse for users over time.

My hypothesis for Anypoint Exchange not seeing any significant improvements is that it didn't provide up-sell opportunities. MuleSoft packages Anypoint Exchange with it's 'Base platform'. It doesn't have any bells and whistles for add-on charges. For advanced use cases around API management, MuleSoft had Anypoint API Community Manager. In the meantime Anypoint Exchange kept on languishing.


Here are my 4 quick ideas to improve Anypoint Exchange's usefulness for it's customers:

1. **Support to store non-Mule assets, relevant for MuleSoft implementation** - 
Anypoint platform currently supports 'MuleSoft assets'. Now what qualifies for 'MuleSoft assets' is debatable. These are connectors, templates, examples, APIs, fragments and other prebuilt assets within MuleSoft ecosystem. You cannot store random jar files in Exchange, even though they could be required for your Mule app to function. This is a significant friction for customers who invested in MuleSoft platform. Many MuleSoft customers aren't traditional Java shops. Customers using .NET ecosystem don't have maven repositories like Jfrog Artifactory or Sonatype Nexus. They have to setup a separate repository to store Mule artifacts, especially when Anypoint Exchange can perfectly help meet this need.

2. **Better content management support** - 
When I was helping Mule customers, I recommended them to use Atlassian Confluence or MediaWiki to write better documentation for their APIs and other guidelines. Anypoint Exchange should be the place to find the documentation for your apis and integration assets. But Anypoint Exchange only works well for simpler documentation needs. It doesn't support mildly complex use cases like resizing of images, or align the content appropriately on a page.

3. **Proxying other Mule repositories** - 
MuleSoft's dependency libraries are distributed via two separate maven compliant repositories.

    a. Public (free) repository - https://repository.mulesoft.org/nexus/content/repositories/releases/

    b. Enterprise (paid) repository - https://repository.mulesoft.org/nexus-ee/content/repositories/releases-ee

    Then there are some other dependencies (e.g. Connectors) distributed via Anypoint Exchange - https://maven.anypoint.mulesoft.com
    And lastly customers have their own private libraries and dependencies based on what they have built so far. 

    These are at least 4 repositories required to build a useful Mule app. Exchange can provide a single proxy to receive the dependency assets to build Mule apps to reduce customer pain.

4. **Secure API Portal** - 
Exchange provides capabilities to build a 'public portal' to publish your APIs and documentation. This unauthenticated portal is publicly available over the internet. Not all customers like their APIs, documentation and request/response examples publicly available. The only alternative is not to publish to a portal. This is not a good developer experience. Exchange should provide a way to setup a private or secure API portal that can solve this problem. 

Addressing these gaps in Anypoint Exchange will improve customer satisfaction and help customers get better return on their MuleSoft investments.
