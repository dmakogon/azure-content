<properties linkid="dev-net-commons-tasks-enable-ssl" urldisplayname="Enable SSL" headerexpose="" pagetitle="Enable SSL - .NET - Develop" metakeywords="Azure SSL, Azure HTTPS, Windows Azure SSL, Windows Azure HTTPS, .NET Azure SSL, .NET Azure HTTPS, C# Azure SSL, C# Azure HTTPS, VB Azure SSL, VB Azure HTTPS" footerexpose="" metadescription="Learn how to specify an HTTPS endpoint for a web role and how to upload an SSL certificate to secure your application." umbraconavihide="0" disquscomments="1"></properties>

<div chunk="../chunks/article-left-menu.md" />
# Configuring SSL for an application in Windows Azure

Secure Socket Layer (SSL) encryption is the most commonly used method of
securing data sent across the internet. This common task discusses how
to specify an HTTPS endpoint for a web role and how to upload an SSL
certificate to secure your application.

This task includes the following steps:

-   [Step 1: Get an SSL certificate][]
-   [Step 2: Modify the service definition and configuration files][]
-   [Step 3: Upload the deployment package and certificate][]
-   [Step 4: Connect to the role instance by using HTTPS][]

This task will use a production deployment; information on using a staging deployment is provided at the end of this topic.

<h2><a name="step1"> </a><span class="short-header">Get an SSL cert</span>Step 1: Get an SSL certificate</h2>

To configure SSL for an application, you first need to get an SSL
certificate that has been signed by a Certificate Authority (CA), a
trusted third-party who issues certificates for this purpose. If you do
not already have one, you will need to obtain one from a company that
sells SSL certificates.

The certificate must meet the following requirements for SSL
certificates in Windows Azure:

-   The certificate must contain a private key.
-   The certificate must be created for key exchange, exportable to a Personal Information Exchange (.pfx) file.
-   The certificate's subject name must match the domain used to access
    the cloud service. You cannot acquire an SSL certificate for the
    cloudapp.net domain, so the certificate's subject name must match
    the custom domain name used to access your application. This documentation uses sslexample.cloudapp.net as the web site URL, so the common name (CN) used in the certificate is **sslexample.cloudapp.net**.
-   The certificate must use a minimum of 2048-bit encryption.

For test purposes, you can create and use a self-signed certificate. For
details about how to create a self-signed certificate using IIS Manager,
See [How to create a certificate for a role][].

Next, you must include information about the certificate in your service
definition and service configuration files.

<h2><a name="step2"> </a><span class="short-header">Modify svc / config files</span>Step 2: Modify the service definition and configuration files</h2>

Your application must be configured to use the certificate, and an HTTPS
endpoint must be added. As a result, the service definition and service
configuration files need to be updated.

1.  In your development environment, open the service definition file
    (CSDEF), add a **Certificates** section within the **WebRole**
    section, and include the following information about the
    certificate:

        <WebRole name="CertificateTesting" vmsize="Small">
        ...
            <Certificates>
                <Certificate name="SampleCertificate" storeLocation="LocalMachine" 
                    storeName="CA" />
            </Certificates>
        ...
        </WebRole>

    The **Certificates** section defines the name of our certificate,
    its location, and the name of the store where it is located. We have
    chosen to store the certificate in the CA (Certificate Authority)
    store, but you can choose other options as well. See [How to
    associate a certificate with a service][] for more information.

2.  In your service definition file, add an **InputEndpoint** element
    within the **Endpoints** section to enable HTTPS:

        <WebRole name="CertificateTesting" vmsize="Small">
        ...
            <Endpoints>
                <InputEndpoint name="HttpsIn" protocol="https" port="443" 
                    certificate="SampleCertificate" />
            </Endpoints>
        ...
        </WebRole>

3.  In your service definition file, add a **Binding** element within
    the **Sites** section. This adds an HTTPS binding to map the
    endpoint to your site:

        <WebRole name="CertificateTesting" vmsize="Small">
        ...
            <Sites>
                <Site name="Web">
                    <Bindings>
                        <Binding name="HttpsIn" endpointName="HttpsIn" />
                    </Bindings>
                </Site>
            </Sites>
        ...
        </WebRole>

    All of the required changes to the service definition file have been
    completed, but you still need to add the certificate information to
    the service configuration file.

4.  In your service configuration file (CSCFG), ServiceConfiguration.Cloud.cscfg, add a **Certificates**
    section within the **Role** section, replacing the sample thumbprint
    value shown below with that of your certificate:

        <Role name="Deployment">
        ...
            <Certificates>
                <Certificate name="SampleCertificate" 
                    thumbprint="9427befa18ec6865a9ebdc79d4c38de50e6316ff" 
                    thumbprintAlgorithm="sha1" />
            </Certificates>
        ...
        </Role>

(The example above uses **sha1** for the thumbprint algorithm. Specify the appropriate value for your certificate's thumbprint algorithm.)

Now that the service definition and service configuration files have
been updated, package your deployment for uploading to Windows Azure. If
you are using **cspack**, ensure that you don't use the
**/generateConfigurationFile** flag, as that will overwrite the
certificate information you just inserted.

<h2><a name="step3"> </a><span class="short-header">Upload to Windows Azure</span>Step 3: Upload the deployment package and certificate</h2>

Your deployment package has been updated to use the certificate, and an
HTTPS endpoint has been added. Now you can upload the package and
certificate to Windows Azure with the Management Portal.

1. Log into the [Windows Azure Management Portal][]. 
2. Click **New**, click **Cloud Service**, and then click **Custom Create**.
3. In the **Create a cloud service** dialog, enter values for the URL, region/affinity group, and subscription. Ensure **Deploy a cloud service package now** is checked, and click the **Next** button.
3. In the **Publish your cloud service** dialog, enter the required information for your cloud service, select **Production** for the environment, and ensure **Add certificates now** is checked. (If any of your roles contain a single instance, ensure **Deploy even if one or more roles contain a single instance** is checked.) 

    ![Publish your cloud service][0]

4.  Click the **Next** button.
5.  In the **Add Certificate** dialog, enter the location for the SSL
    certificate .pfx file, the password for the certificate, and click
    **attach certificate**.  

    ![Add certificate][1]

6.  Ensure your certificate is listed in the **Attached Certificates** section.

    ![Attached certificates][4]

7.  Click the **Complete** button to create your cloud service. When the deployment has reached the **Ready** status, you can proceed to the next steps.

<h2><a name="step4"> </a><span class="short-header">Connect using HTTPS</span>Step 4: Connect to the role instance by using HTTPS</h2>

Now that your deployment is up and running in Windows Azure, you can
connect to it using HTTPS.

1.  In the Management Portal, select your deployment, then click the link under **Site URL**.

    ![Determine site URL][2]

2.  In your web browser, modify the link to use **https** instead of **http**, and then visit the page.

    **Note:** If you are using a self-signed certificate, when you
    browse to an HTTPS endpoint that's associated with the self-signed
    certificate you will see a certificate error in the browser. Using a
    certificate signed by a trusted certification authority will eliminate this problem; in the meantime, you can ignore the error. (Another option is to add the self-signed certificate to the user's trusted certificate authority certificate store.)

    ![SSL example web site][3]

If you want to use SSL for a staging deployment instead of a production deployment, you'll first need to determine the URL used for the staging deployment. Deploy your cloud service to the staging environment without including a certificate or any certificate information. Once deployed, you can determine the GUID-based URL, which is listed in the management portal's **Site URL** field. Create a certificate with the common name (CN) equal to the GUID-based URL (for example, **32818777-6e77-4ced-a8fc-57609d404462.cloudapp.net**), use the management portal to add the certificate to your staged cloud service, add the certificate information to your CSDEF and CSCFG files, repackage your application, and update your staged deployment to use the new package and CSCFG file.

<h2><a name="additional_resources"> </a><span class="short-header">Additional Resources</span>Additional Resources</h2>

* [How to associate a certificate with a service][]

* [How to configure an SSL certificate on an HTTPS endpoint][]

  [Step 1: Get an SSL certificate]: #step1
  [Step 2: Modify the service definition and configuration files]: #step2
  [Step 3: Upload the deployment package and certificate]: #step3
  [Step 4: Connect to the role instance by using HTTPS]: #step4
  [How to create a certificate for a role]: http://msdn.microsoft.com/en-us/library/windowsazure/gg432987.aspx
  [How to associate a certificate with a service]: http://msdn.microsoft.com/en-us/library/windowsazure/gg465718.aspx
  [Windows Azure Management Portal]: http://manage.windowsazure.com
  [0]: ../../../DevCenter/Shared/Media/CreateCloudService.png
  [1]: ../../../DevCenter/Shared/Media/AddCertificate.png
  [2]: ../../../DevCenter/Shared/Media/CopyURL.png
  [3]: ../../../DevCenter/Shared/Media/SSLCloudService.png
  [4]: ../../../DevCenter/Shared/Media/AddCertificateComplete.png  
  [How to configure an SSL certificate on an HTTPS endpoint]: http://msdn.microsoft.com/en-us/library/windowsazure/ff795779.aspx
