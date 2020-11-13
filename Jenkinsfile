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
		pip install --user sphinx
		pip install --user sphinx_rtd_theme
		export PATH=${PATH}:~/.local/bin
		make html
		make html
		sed -i -r "/\\[NULL\\]/d" build/html/*html # Remove dead links
		'''
	    }
	}
	stage('Store') {
	    steps {
		sh '''
		rm -rf ../stash
		mkdir ../stash
		mv build ../stash/
		'''
	    }
	}
	stage('Deploy') {
	    steps {
		withCredentials([string(credentialsId: 'GitHub_API_Token',
					variable: 'api_token')]) {
		sh '''
		git checkout gh-pages
		git rm -rf stable \${HORACE_VERSION}
		echo "Bypassing Jekyll on GitHub Pages" > .nojekyll
		git add .nojekyll
		cp -r ../stash/build/html \${HORACE_VERSION}
		git add *
		git commit -m "Document build from CI"
		git push origin gh-pages
		'''
		}
	    }
	}
    }
}
