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
            tower-cli job launch --job-template "Test - $item check" --monitor >$item.output
            OK=$(cat $item.output|grep unreachable|awk '{ print $3 }'|cut -d= -f2)
            CHANGED=$(cat $item.output|grep unreachable|awk '{ print $4 }'|cut -d= -f2)
        	UNREACHABLE=$(cat $item.output|grep unreachable|awk '{ print $5 }'|cut -d= -f2)
            FAILED=$(cat $item.output|grep unreachable|awk '{ print $5 }'|cut -d= -f2)
            echo "OK=$OK"
            echo "CHANGED=$CHANGED"
            echo "UNREACHABLE=$UNREACHABLE"
            echo "FAILED=$FAILED"
            echo "Test run: $item"
        done
        '''
    }
    stage("Create job template") {
        sh '''
        for item in ${TEMPLATES[@]}; do
            echo "Create job template: $item"
        done
        '''
    }
}
