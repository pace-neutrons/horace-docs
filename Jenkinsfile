#!groovy

// git clone https://pace-builder:%api_token%@github.com/pace-neutrons/horace-docs . &

pipeline {
    agent {
	label 'PACE Windows (Private)'
    }
    stages {
	stage('Checkout') {
	    steps {
		withCredentials([string(credentialsId: 'GitHub_API_Token',
			    variable: 'api_token')]) {
		    bat '''
			git config --local user.name "PACE CI Build Agent" &
		    '''
		}
	    }
	}
	stage ('Prepare') {
	    steps {
		bat '''
		    git checkout %BRANCH_NAME%
		'''
	    }
	}
	stage('Build') {
	    steps {
		bat '''
		    pip install sphinx &
		    pip install sphinx_rtd_theme &
		    make.bat html &
		    make.bat html
		'''
	    }
	}
	stage('Store') {
	    steps {
		bat '''
		    rmdir /S /Q ..\\stash &
		    mkdir ..\\stash &
		    move build ..\\stash
		'''
	    }
	}
	stage('Deploy') {
	    steps {
		bat '''
		    git checkout gh-pages &
		    git rm -rf .
		    echo "Bypassing Jekyll on GitHub Pages" > .nojekyll &
		    git add .nojekyll &
		    robocopy /E /NFL /NDL /NJS /nc /ns /np ..\\stash\\build\\html . &
		    robocopy /E /NFL /NDL /NJS /nc /ns /np ..\\stash\\build\\html %HORACE_VERSION% &
		    git add * &
		    git commit -m "Document build from CI" &
		    git push origin gh-pages
		'''
	    }
	}
    }
}
