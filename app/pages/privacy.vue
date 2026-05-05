<script setup lang="ts">
defineI18nRoute(false)

useHead({
  title: 'Privacy Policy | Freetool.Dev',
  meta: [{ name: 'description', content: 'FreeTool.dev Privacy and Cookies Policy — learn how we handle your data, cookies, and third-party services.' }],
})

usePageJsonLd({
  type: 'page',
  name: 'Privacy Policy | Freetool.Dev',
})
</script>

<template>
  <div class="container mx-auto px-4 py-24 min-h-[60vh]">
    <div class="max-w-3xl mx-auto">
      <h1 class="text-4xl font-extrabold mb-2 bg-gradient-to-r from-primary to-secondary bg-clip-text text-transparent">
        Privacy &amp; Cookies Policy
      </h1>
      <p class="text-sm opacity-50 mb-10">Last updated: May 5, 2026</p>

      <div class="space-y-8 opacity-85 leading-relaxed">

        <section>
          <h2 class="text-lg font-bold mb-2 text-orange-300">1. Introduction</h2>
          <p>
            FreeTool.dev ("<strong>we</strong>", "<strong>our</strong>", or "<strong>us</strong>") is committed to protecting your privacy.
            This Privacy and Cookies Policy explains what information we collect when you visit
            <strong>freetool.dev</strong> (the "Site"), how we use it, and the choices available to you.
            By continuing to use the Site, you acknowledge and agree to the practices described in this Policy.
          </p>
        </section>

        <section>
          <h2 class="text-lg font-bold mb-2 text-orange-300">2. Information We Collect</h2>
          <p>We do <strong>not</strong> require you to create an account or provide any personal information to use our tools.
          The information collected automatically is limited to:</p>
          <ul class="list-disc list-inside mt-2 space-y-1 opacity-90">
            <li>Anonymized and truncated IP addresses (processed by Google Analytics).</li>
            <li>Browser type, device type, and operating system.</li>
            <li>Pages visited, time spent on each page, and referring URLs.</li>
            <li>General geographic region (country/city level, derived from truncated IP).</li>
          </ul>
          <p class="mt-2">
            This information is collected in aggregate form and cannot be used to personally identify you.
          </p>
        </section>

        <section>
          <h2 class="text-lg font-bold mb-2 text-orange-300">3. Cookies and Tracking Technologies</h2>
          <p>
            We use cookies and similar tracking technologies to analyze Site usage and serve relevant advertising.
            The types of cookies we use are:
          </p>
          <ul class="list-disc list-inside mt-2 space-y-1 opacity-90">
            <li><strong>Analytics cookies</strong> — collect anonymous data about how visitors interact with the Site.</li>
            <li><strong>Advertising cookies</strong> — used by our advertising partners to show relevant ads.</li>
          </ul>
          <p class="mt-2">
            We do <strong>not</strong> use cookies to collect personally identifiable information, and we do not sell your data to third parties.
          </p>
        </section>

        <section>
          <h2 class="text-lg font-bold mb-2 text-orange-300">4. Google Analytics</h2>
          <p>
            This Site uses <strong>Google Analytics</strong>, a web analytics service provided by Google LLC ("Google").
            Google Analytics uses performance cookies to collect anonymized browsing data (including truncated IP addresses).
            This data is used exclusively to analyze Site traffic, compile usage reports, and improve our services.
            IP anonymization is enabled — your full IP address is never stored by Google on our behalf.
          </p>
          <p class="mt-2">For more information, please review:</p>
          <ul class="list-disc list-inside mt-1 space-y-1">
            <li>
              <a href="https://policies.google.com/technologies/cookies" rel="noopener noreferrer" target="_blank" class="link">
                How Google uses cookies
              </a>
            </li>
            <li>
              <a href="https://tools.google.com/dlpage/gaoptout" rel="noopener noreferrer" target="_blank" class="link">
                Google Analytics Opt-out Browser Add-on
              </a>
            </li>
            <li>
              <a href="https://policies.google.com/technologies/partner-sites" rel="noopener noreferrer" target="_blank" class="link">
                Google Privacy &amp; Terms
              </a>
            </li>
          </ul>
        </section>

        <section>
          <h2 class="text-lg font-bold mb-2 text-orange-300">5. Advertising — Google AdSense &amp; DoubleClick DART Cookie</h2>
          <p>
            We use <strong>Google AdSense</strong> to display advertisements on the Site. Google and its partners may use
            DART cookies to serve ads based on your visits to this Site and other sites on the Internet.
            Our Google AdSense account is configured to comply with <strong>GDPR</strong> and <strong>CCPA</strong> requirements,
            including support for non-personalized ads where applicable.
          </p>
          <p class="mt-2">You can manage or opt out of personalized advertising at any time:</p>
          <ul class="list-disc list-inside mt-1 space-y-1">
            <li>
              <a href="https://adssettings.google.com/u/0/authenticated" rel="noopener noreferrer" target="_blank" class="link">
                Google Ad Settings
              </a>
            </li>
            <li>
              <a href="https://policies.google.com/technologies/ads" rel="noopener noreferrer" target="_blank" class="link">
                Google Ad Policies
              </a>
            </li>
          </ul>
        </section>

        <section>
          <h2 class="text-lg font-bold mb-2 text-orange-300">6. Disabling or Managing Cookies</h2>
          <p>
            You can control and manage cookies directly in your browser settings. Most browsers allow you to refuse new cookies,
            delete existing cookies, or be notified when a new cookie is set.
            Please note that disabling certain cookies may affect the functionality of some Site features.
          </p>
          <p class="mt-2">
            Residents of the European Union may also use the
            <a href="https://www.youronlinechoices.eu/" rel="noopener noreferrer" target="_blank" class="link">Your Online Choices</a>
            platform to manage behavioral advertising preferences.
            California residents may exercise their rights under CCPA by opting out of the "sale" of personal information
            via the Google Ad Settings link above (we do not sell personal data directly).
          </p>
        </section>

        <section>
          <h2 class="text-lg font-bold mb-2 text-orange-300">7. Data Processing &amp; Tool Privacy</h2>
          <p>
            All interactive tools available on FreeTool.dev operate <strong>entirely within your browser</strong>.
            No input data — text, files, or any other content you paste or upload — is transmitted to our servers,
            stored, logged, or shared with any third party.
            Your data never leaves your device while using our tools.
          </p>
        </section>

        <section>
          <h2 class="text-lg font-bold mb-2 text-orange-300">8. External Links</h2>
          <p>
            The Site may contain links to third-party websites. We are not responsible for the content, privacy practices,
            or security of those sites. We encourage you to read the privacy policies of any external site you visit,
            as their practices may differ significantly from ours.
          </p>
        </section>

        <section>
          <h2 class="text-lg font-bold mb-2 text-orange-300">9. Children's Privacy</h2>
          <p>
            FreeTool.dev is not directed at children under the age of 13. We do not knowingly collect any personal
            information from children. If you believe a child has provided us with personal data, please contact us
            so we can take appropriate action.
          </p>
        </section>

        <section>
          <h2 class="text-lg font-bold mb-2 text-orange-300">10. Your Rights</h2>
          <p>Depending on your jurisdiction, you may have the right to:</p>
          <ul class="list-disc list-inside mt-2 space-y-1 opacity-90">
            <li>Access the personal data we hold about you.</li>
            <li>Request correction or deletion of your data.</li>
            <li>Object to or restrict certain types of data processing.</li>
            <li>Withdraw consent at any time (e.g., by disabling cookies in your browser).</li>
            <li>Lodge a complaint with your local data protection authority.</li>
          </ul>
          <p class="mt-2">
            Because we collect only anonymized analytics data through Google Analytics and do not maintain user accounts,
            most of these rights are exercised directly through your browser or Google's own privacy controls linked above.
          </p>
        </section>

        <section>
          <h2 class="text-lg font-bold mb-2 text-orange-300">11. Changes to This Policy</h2>
          <p>
            We reserve the right to update this Privacy Policy periodically to reflect changes in our practices or
            applicable law. When we do, we will revise the "Last updated" date at the top of this page.
            Continued use of the Site after any changes constitutes your acceptance of the updated Policy.
            We encourage you to review this page periodically.
          </p>
        </section>

        <section>
          <h2 class="text-lg font-bold mb-2 text-orange-300">12. Contact</h2>
          <p>
            If you have any questions or concerns about this Privacy Policy or our data practices, please reach out to us
            through the contact information available on the Site. We will do our best to respond promptly.
          </p>
        </section>

      </div>
    </div>
  </div>
</template>
