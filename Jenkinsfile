pipeline {
    agent {
        docker { image 'lochnair/octeon-buildenv:latest' }
    }

    stages {
        stage('Clean') {
            steps {
               sh 'git reset --hard'
               sh 'git clean -fX'
            }
        }

        stage('Prepare for out-of-tree builds') {
            steps {
                sh 'make -j5 ARCH=mips CROSS_COMPILE=mips64-octeon-linux- prepare modules_prepare'
                sh 'rm -rf tmp && mkdir tmp'
                sh 'tar --exclude-vcs --exclude=tmp -cf tmp/e300-ksrc.tar .'
                sh 'mv tmp/e300-ksrc.tar e300-ksrc.tar'
            }
        }

        stage('Build') {
            steps {
                sh 'make -j5 ARCH=mips CROSS_COMPILE=mips64-octeon-linux- vmlinux modules'
            }
        }
        
        stage('Archive kernel image') {
            steps {
                sh 'mv -v vmlinux vmlinux.64'
                sh 'tar cvjf e300-kernel.tar.bz2 vmlinux.64'
                archiveArtifacts artifacts: 'e300-kernel.tar.bz2', fingerprint: true, onlyIfSuccessful: true
            }
        }
        
        stage('Archive kernel modules') {
            steps {
                sh 'make ARCH=mips CROSS_COMPILE=mips64-octeon-linux- INSTALL_MOD_PATH=destdir modules_install'
                sh 'tar cvjf e300-modules.tar.bz2 -C destdir .'
                archiveArtifacts artifacts: 'e300-modules.tar.bz2', fingerprint: true, onlyIfSuccessful: true
            }
        }

        stage('Archive build tree') {
            steps {
                sh 'tar -uvf e300-ksrc.tar Module.symvers'
                sh 'bzip2 e300-ksrc.tar'
                archiveArtifacts artifacts: 'e300-ksrc.tar.bz2', fingerprint: true, onlyIfSuccessful: true
            }
        }
    }
}
