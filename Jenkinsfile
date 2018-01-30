node {
    git url: 'https://github.com/mglantz/tomcat-playbook.git', branch: 'bugfix'
    stage("Syntax check") {
        sh '''
        echo "Syntax checking playbook"
        # Run a syntax check for the playbook
        ansible-playbook --syntax-check ./tomcat.yml
        '''
    }
    stage("AWX Project refresh") {
        sh '''
        # This ensures that Ansible Tower has the latest version of playbooks in our project
        echo "Refreshing project in Ansible Tower."
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
        tower-cli job launch --job-template "Test - tomcat" --monitor >tomcat.output || true
        OK=$(cat tomcat.output|grep unreachable|awk '{ print $3 }'|cut -d= -f2)
        CHANGED=$(cat tomcat.output|grep unreachable|awk '{ print $4 }'|cut -d= -f2)
        UNREACHABLE=$(cat tomcat.output|grep unreachable|awk '{ print $5 }'|cut -d= -f2)
        FAILED=$(cat tomcat.output|grep unreachable|awk '{ print $5 }'|cut -d= -f2)
        if [ "$UNREACHABLE" -ne 0 ] && [ "$FAILED" -ne 0 ]; then
            echo "Failed test run with errors. Task status summary:"
            echo "OK=$OK"
            echo "CHANGED=$CHANGED"
            echo "UNREACHABLE=$UNREACHABLE"
            echo "FAILED=$FAILED"
            echo "Job output:"
            cat tomcat.output
            exit 1
        else
            echo "Test run successful for: Test - tomcat"
        fi
        '''
    }
    stage("Test Tomcat application") {
        sh '''
        # Test if we can reach the Tomcat application
        # First we need to fetch a list of hosts in our inventory
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
        input "Please go ahead and merge your changes to the master branch"
    }
    stage("Create job template") {
        sh '''
        # Create a job template in Tower and set label indicating that it has been tested OK
        tower-cli job_template create --name "Tomcat" --description "Created by Jenkins: $(date)" --job-type run --inventory Hostnetwork --project "Tomcat Playbooks" --playbook "tomcat.yml" --credential "Required access on hostnet"
        tower-cli label create -n testing_ok --organization Default --job-template Tomcat
        '''
    }
}
