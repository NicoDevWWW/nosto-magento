#!/usr/bin/env groovy

node {
    stage "Prepare environment"
        checkout scm
        def environment  = docker.build 'platforms-base'

        environment.inside {
            stage "Update Dependencies"
                sh "composer install"
                sh "composer dump-autoload --optimize"
                sh "./vendor/bin/pearify process ."

            stage "Code Sniffer"
                catchError {
                    sh "./vendor/bin/phpcbf --standard=ruleset.xml app"
                    sh "./vendor/bin/phpcs --standard=ruleset.xml --severity=10 --report=checkstyle --report-file=phpcs.xml app lib || true"
                }
                step([$class: 'hudson.plugins.checkstyle.CheckStylePublisher', pattern: 'phpcs.xml', unstableTotalAll:'0'])

            stage "Copy-Paste Detection"
                sh "./vendor/bin/phing phpcpd"

            stage "Mess Detection"
                catchError {
                    sh "./vendor/bin/phpmd . xml codesize,naming,unusedcode,controversial,design --exclude vendor,var,build,tests --reportfile phpmd.xml"
                }
                //step([$class: 'PmdPublisher', pattern: 'phpmd.xml', unstableTotalAll:'0'])

            stage "Phan Analysis"
                catchError {
                    sh "./vendor/bin/phan --signature-compatibility --config-file=phan.php"
                }
                step([$class: 'hudson.plugins.checkstyle.CheckStylePublisher', pattern: 'phpcs.xml', unstableTotalAll:'0'])

        }

    stage "Cleanup"
        deleteDir()
}
