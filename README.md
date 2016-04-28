nginx proxy for serving gunicorn etc from Kubernetes
====================================================

This container provides an nginx service that proxies connections from
port 80 to port 8000 on localhost. In addition it also serves the directories
`/mnt/media` and `/mnt/static` as `/media` and `/static` respectively.

The intention is to use this container to serve gunicorn (etc.) from inside
a Kubernetes pod.

The nginx-proxy can optionally take a file `/etc/secrets/nginx-proxy/htaccess`
which if it exists will apply HTTP basic auth to the site.

An example Kubernetes/Openshift deployment is here:

    kind: DeploymentConfig
    apiVersion: v1
    spec:
      template:
        spec:
          containers:
           - image: web
             name: web
             volumeMounts:
              - name: static
                mountPath: /mnt/static

           - image: nginx-proxy
             name: nginx-proxy
             volumeMounts:
              - name: static
                mountPath: /mnt/static

              - name: nginx-proxy
                mountPath: /etc/secrets/nginx-proxy

          volumes:
           - name: static
             emptyDir: {}

           - name: nginx-proxy
             secret:
               secretName: nginx-proxy

To deploy your static content into /mnt/static do it when running the app.
Expose gunicorn on port 8000.

    EXPOSE 8000
    CMD cp -rv /app/static/* /mnt/static/ && \
        gunicorn \
            -w 4 \
            -b 0.0.0.0:8000 \
            myapp.wsgi:application

You can create a htpassword file with:

    htpasswd -c htacesss <user>
    oc secrets new nginx-proxy htaccess=htaccess

Or leave it out with:

    oc secrets new nginx-proxy nothing=/dev/null
