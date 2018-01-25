node {
    git url: 'https://github.com/mglantz/tomcat-playbook.git'
    stage("Syntax check") {
        sh '''
        echo "Syntax checking playbooks"
        TEMPLATES=$(ls|grep yml|cut -d'.' -f1)
        # For all playbooks files we find in the cloned git repository
        for item in ${TEMPLATES[@]}; do
            # Run a syntax check for all the playbooks
            ansible-playbook --syntax-check ./$item.yml
        done
        '''
    }
    stage("AWX Project refresh") {
        sh '''
        # This ensures that Ansible Tower has the latest version of any playbooks
        echo "Refreshing project in Ansible Tower."
        tower-cli project update -n "Tomcat Playbooks" --monitor
        '''
    }
    stage("Test run") {
        sh '''
        TEMPLATES=$(ls|grep yml|cut -d'.' -f1)
        for item in ${TEMPLATES[@]}; do
            # Delete previously created test playbooks
            if tower-cli job_template list|grep "Test - $item" >/dev/null; then
                echo "Found existing Test job_template for $item. Deleting it."
                tower-cli job_template delete --name "Test - $item check"
            fi
            echo "Creating job_template: Test - $item"
            tower-cli job_template create --name "Test - $item check" --description "Created by Jenkins: $(date)" --job-type run --inventory Hostnetwork --project "Tomcat Playbooks" --playbook "$item.yml" --credential "Required access on hostnet" --verbosity "debug"
            tower-cli job launch --job-template "Test - $item check" --monitor >$item.output || true
            OK=$(cat $item.output|grep unreachable|awk '{ print $3 }'|cut -d= -f2)
            CHANGED=$(cat $item.output|grep unreachable|awk '{ print $4 }'|cut -d= -f2)
        	UNREACHABLE=$(cat $item.output|grep unreachable|awk '{ print $5 }'|cut -d= -f2)
            FAILED=$(cat $item.output|grep unreachable|awk '{ print $5 }'|cut -d= -f2)
            if [ "$UNREACHABLE" -ne 0 ] && [ "$FAILED" -ne 0 ]; then
                echo "Failed test run with errors. Task status summary:"
                echo "OK=$OK"
                echo "CHANGED=$CHANGED"
                echo "UNREACHABLE=$UNREACHABLE"
                echo "FAILED=$FAILED"
                echo "Job output:"
                cat $item.output
                exit 1
            else
                echo "Test run successful for: $item"
            fi
        done
        '''
    }
    stage("Create job template") {
        sh '''
        for item in ${TEMPLATES[@]}; do
            if tower-cli job_template list|grep "$item" >/dev/null; then
                echo "Found existing job_template for $item. Doing nothing."
            else
                tower-cli job_template create --name "Test - $item check" --description "Created by Jenkins: $(date)" --job-type run --inventory Hostnetwork --project "Tomcat Playbooks" --playbook --job-tags testing_ok
                if [ "$?" -eq 0 ]; then
                    echo "Created template for: $item"
                else
                    echo "Failed to create template for: $item"
                fi
            fi
        done
        '''
    }
}
