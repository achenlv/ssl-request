#!/usr/bin/env groovy

pipeline {
  parameters {
    string(name: 'DOMAIN', defaultValue: params.DOMAIN?:'some.domain.dlt', description: 'Common Domain Name, no wildcard')
    choice(name: 'WILDCARD', choices: ['NO', 'YES'], description: 'Use YES for wildcard certificate.')
    string(name: 'COUNTRY', defaultValue: params.COUNTRY?:'US', description: 'Country Name')
    string(name: 'STATE', defaultValue: params.STATE?:'ILLINOIS', description: 'State or Province Name')
    string(name: 'CITY', defaultValue: params.CITY?:'PARK RIDGE', description: 'City Name')
    string(name: 'ORGANIZATION', defaultValue: params.ORGANIZATION?:'Accenture LLP', description: 'Organization Name')
    string(name: 'EMAIL', defaultValue: params.EMAIL?:'mywizard.dmt.platform.ext.support@accenture.com')
  }
  
  environment {
    REQ_CONFIG_FILE = "./${params.DOMAIN}.cnf"
    DOMAIN = "${params.DOMAIN}"
    COUNTRY = "${params.COUNTRY}"
    STATE = "${params.STATE}"
    CITY = "${params.CITY}"
    ORGANIZATION = "${params.ORGANIZATION}"
    EMAIL = "${params.EMAIL}"
    WILDCARD = "${params.WILDCARD}"
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
        // checkout scm
        sh """
          echo $DOMAIN
          echo Installed OpenSSL: \$(openssl version)
        """
      }
    }
    
    stage('Prepare') {
      steps {
        sh """
          echo "[ req ]" > ${REQ_CONFIG_FILE}
          echo "prompt = no" >> ${REQ_CONFIG_FILE}
          echo "default_bits = 2048" >> ${REQ_CONFIG_FILE}
          echo "distinguished_name = req_distinguished_name" >> ${REQ_CONFIG_FILE}
          [[ "${WILDCARD}" == "YES" ]] \
              && echo "req_extensions = req_ext" >> ${REQ_CONFIG_FILE}
          echo "[ req_distinguished_name ]" >> ${REQ_CONFIG_FILE}
          echo "countryName = ${COUNTRY}" >> ${REQ_CONFIG_FILE}
          echo "stateOrProvinceName = ${STATE}" >> ${REQ_CONFIG_FILE}
          echo "localityName = ${CITY}" >> ${REQ_CONFIG_FILE}
          echo "organizationName = ${ORGANIZATION}" >> ${REQ_CONFIG_FILE}
          echo "commonName = ${DOMAIN}" >> ${REQ_CONFIG_FILE}
          echo "emailAddress = ${EMAIL}" >> ${REQ_CONFIG_FILE}
          [[ "${WILDCARD}" == "YES" ]] \
              && echo "[ req_ext ]" >> ${REQ_CONFIG_FILE} \
              && echo "subjectAltName = @alt_names" >> ${REQ_CONFIG_FILE} \
              && echo "[ alt_names ]" >> ${REQ_CONFIG_FILE} \
              && echo "DNS.1 = ${DOMAIN}" >> ${REQ_CONFIG_FILE} \
              && echo "DNS.2 = *.${DOMAIN}" >> ${REQ_CONFIG_FILE}
        """
      }
    }

    stage('Generate') {
      steps {
        sh """
          openssl req -out ${DOMAIN}.csr -newkey rsa:2048 -nodes -keyout ${DOMAIN}.key -config ${REQ_CONFIG_FILE}
        """
      }
    }

    stage('Smoke Test') {
      steps {
        echo "[TEST] Testing request sanity"
        sh """
          openssl req -text -noout -verify -in ${REQ_CONFIG_FILE}
        """
        echo "[TEST] Testing private key"
        sh """
          openssl rsa -text -noout -in ${DOMAIN}.key
        """
      }
    }
  }
  post {
    success {
      // Can save to AWS SecretsManager directly
      archiveArtifacts artifacts: "${env.DOMAIN}.*", onlyIfSuccessful: true
    }
  }
}