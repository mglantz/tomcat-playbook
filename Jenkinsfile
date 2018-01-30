node {
    git url: 'https://github.com/mglantz/tomcat-playbook.git', branch: 'bugfix'
    stage("Syntax check") {
        sh '''
        echo "Syntax checking playbook"
        # Run a syntax check for the playbook
        ansible-playbook --syntax-check ./tomcat.yml
        '''
    }
    stage("Style check") {
        sh '''
        echo "Style checking playbook"
        TEST=$(ansible-lint ./tomcat.yml)
        if [ "$TEST" == "" ]; then
            echo "Style check OK"
        else
            echo "Style check issues:"
            ansible-lint ./tomcat.yml
            exit 1
        fi
        '''
    }
    stage("AWX Project refresh") {
        sh '''
        # This ensures that Ansible Tower has the latest version of playbooks in our project
        echo "Refreshing project in Ansible Tower from which connects to our dev branch and playbook."
        tower-cli project update -n "Tomcat Playbooks Dev" --monitor
        '''
    }
    stage("Test run") {
        sh '''
        # Do a test run of the playbook
        if tower-cli job_template list|grep "Test - tomcat" >/dev/null; then
            echo "Found existing Test job_template for tomcat. Deleting it."
            tower-cli job_template delete --name "Test - tomcat"
        fi
        echo "Creating job_template: Test - tomcat"
        tower-cli job_template create --name "Test - tomcat" --description "Created by Jenkins: $(date)" --job-type run --inventory Hostnetwork --project "Tomcat Playbooks Dev" --playbook "tomcat.yml" --credential "Required access on hostnet" --verbosity "debug"
        
        # Launch run of job template to see if it works. Force exit to 0 and evaluate
        tower-cli job launch --job-template "Test - tomcat" --monitor >tomcat.output || true
        if grep -q 'unreachable=0.*failed=0' tomcat.output; then
            echo "Test run successful for: Test - tomcat"
        else
            echo "Failed test. Playbook output:"
            cat tomcat.output
            exit 1
        fi
        '''
    }
    stage("Test idempotence") {
        sh '''
        # Additional run of playbook to test playbook idempotence
        tower-cli job launch --job-template "Test - tomcat" --monitor >tomcat.output || true
        
        if grep -q 'changed=0.*unreachable=0.*failed=0' tomcat.output; then
            echo "Idempotence test OK"
        else
            echo "Idempotence test failed. Details below:"
            cat tomcat.output
            exit 1
        fi
        '''
    }
    stage("Test Tomcat application") {
        sh '''
        # Test if we can reach the Tomcat application
        # First we need to fetch a list of hosts in our inventory from Ansible Tower
        HOSTS=$(tower-cli host list -i 5|grep local.net|awk '{ print $2 }')
        FAIL=0
        for item in ${HOSTS[@]}; do
                if curl -v http://$item:8080/sample/hello.jsp|grep "This is the output of a JSP page" >/dev/null; then
                    echo "Got OK answer from Tomcat running on: $item"
                    echo "Deleting test template"
                    tower-cli job_template delete --name "Test - tomcat"
                else
                    echo "Did not get answer from Tomcat running on: $item"
                    exit 1
                fi
        done
        '''
    }
    stage("Merge to master") {
        input "Press proceed when you have merged the code to master"
    }
    stage("Create job template") {
        sh '''
        # Create a job template in Tower and set label indicating that it has been tested OK
        tower-cli job_template create --name "Tomcat" --description "Created by Jenkins: $(date)" --job-type run --inventory Hostnetwork --project "Tomcat Playbooks" --playbook "tomcat.yml" --credential "Required access on hostnet"
        tower-cli label create -n testing_ok --organization Default --job-template Tomcat
        '''
    }
}
