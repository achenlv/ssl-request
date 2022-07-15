#!/usr/bin/env groovy

pipeline {
  parameters {
    string(name: 'DOMAIN', defaultValue: params.DOMAIN?:'some.domain.dlt', description: 'Common Domain Name, no wildcard')
    string(name: 'COUNTRY', defaultValue: params.COUNTRY?:'US', description: 'Country Name')
    string(name: 'STATE', defaultValue: params.STATE?:'ILLINOIS', description: 'State or Province Name')
    string(name: 'CITY', defaultValue: params.CITY?:'PARK RIDGE', description: 'City Name')
    string(name: 'ORGANIZATION', defaultValue: params.ORGANIZATION?:'Accenture LLP', description: 'Organization Name')
    string(name: 'EMAIL', defaultValue: params.EMAIL?:'mywizard.dmt.platform.ext.support@accenture.com')
  }
  
  environment {
    DOMAIN = "${params.DOMAIN}"
    COUNTRY = "${params.COUNTRY}"
    STATE = "${params.STATE}"
    ORGANIZATION = "${params.ORGANIZATION}"
    EMAIL = "${params.EMAIL}"
  }
  
  agent {
    label 'k8s-jnlp-agent'
  }
  
  options {
    disableConcurrentBuilds()
    skipDefaultCheckout()
    timestamps()
    ansiColor('xterm')
  }

  stages {
    stage('Prerequisites') {
      steps {
        sh """
          echo $DOMAIN
          echo Installed OpenSSL: \$(openssl version)
        """
      }
    }
    
    stage('Prepare') {
      steps {
        sh """
          cat <<EOF > san.cnf
          [ req ]
          prompt = no
          default_bits = 2048
          distinguished_name = req_distinguished_name
          req_extensions     = req_ext
          [ req_distinguished_name ]
          countryName = ${COUNTRY}
          stateOrProvinceName = ${STATE}
          localityName = ${CITY}
          organizationName = ${ORGANIZATION}
          commonName = ${DOMAIN}
          emailAddress = ${EMAIL}
          [ req_ext ]
          subjectAltName = @alt_names
          [ alt_names ]
          DNS.1 = ${DOMAIN}
          DNS.2 = *.${DOMAIN}         
          EOF
          pwd
          ls -la
          cat san.cnf 
        """
      }
    }

    stage('Generate') {
      steps {
        sh """
          # openssl req -out sslcert.csr -newkey rsa:2048 -nodes -keyout private.key -config san.cnf
        """
      }
    }
  }
  post {
    success {
      // archiveArtifacts artifacts: 'private.key, sslcert.csr', onlyIfSuccessful: true
    }
  }
}