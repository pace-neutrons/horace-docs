#!groovy

pipeline {
    agent {
	label 'sl7'
    }
    stages {
	stage('Checkout') {
	    steps {
		withCredentials([string(credentialsId: 'GitHub_API_Token',
					variable: 'api_token')]) {
		    sh '''
		    git config --local user.name "PACE CI Build Agent"
		    '''
		}
	    }
	}
	stage ('Prepare') {
	    steps {
		sh '''
		git checkout \${BRANCH_NAME}
		'''
	    }
	}
	stage('Build') {
	    steps {
		sh '''
		module load python/3.6
		pip install sphinx
		pip install sphinx_rtd_theme
		make html
		make html
		sed -i -r "/\\[NULL\\]/d" \${HORACE_VERSION}/build/html/*html # Remove dead links
		'''
	    }
	}
	stage('Store') {
	    steps {
		sh '''
		rm -rf ../stash/*
		mv build ../stash/
		'''
	    }
	}
	stage('Deploy') {
	    steps {
		sh '''
		git checkout gh-pages
		git rm -rf stable \${HORACE_VERSION}
		echo "Bypassing Jekyll on GitHub Pages" > .nojekyll
		git add .nojekyll
		cp ../stash/build/html \${HORACE_VERSION}
		git add *
		git commit -m "Document build from CI"
		git push origin gh-pages
		'''
	    }
	}
    }
}
