require 'rdoc/task'

versions = {
  "master" => "master",
  "3.2" => "ruby_3_2",
  "3.1" => "ruby_3_1",
  "3.0" => "ruby_3_0",
  "2.7.0" => "ruby_2_7",
}

def sh_with_unbundled_env(...)
  Bundler.with_unbundled_env do
    sh(...)
  end
end

versions.each do |version, branch_name|
  source_dir = "sources/#{version}"

  directory source_dir do
    sh "git clone --depth=1 --branch=#{branch_name} https://github.com/ruby/ruby #{source_dir}"
  end

  desc "Checks out source for #{version}"
  task "source:#{version}" => source_dir

  desc "Updates source for #{version}"
  task "update:#{version}" => source_dir do
    Dir.chdir source_dir do
      sh "git fetch origin"
      sh "git reset origin/#{branch_name} --hard"
    end
  end

  desc "Compile source for #{version}"
  task "compile:#{version}" => source_dir do
    Dir.chdir source_dir do
      sh "autoconf && ./configure --disable-install-doc" unless File.exist?("Makefile")
      sh_with_unbundled_env "make -j2"
    end
  end

  namespace :rdoc do
    lang_version = File.join("en", version)
    task lang_version => "compile:#{version}" do
      sh_with_unbundled_env(
        "make", "html",
        "RDOCOPTS=--title=\"Documentation for Ruby #{version}\"" \
        "--main=README.md",
        "HTMLOUT=#{Dir.pwd}/#{version}",
        chdir: source_dir
      )
    end
  end
  task "rdoc:#{version}" => ["update:#{version}", "compile:#{version}"]
end

desc "Checks out sources for all versions"
task "source" => versions.keys.map { |version| "source:#{version}" }

desc "Updates sources for all versions"
task "update" => versions.keys.map { |version| "update:#{version}" }

desc "Build RDoc HTML files for all versions"
task "rdoc" => versions.keys.map { |version| "rdoc:#{version}" }
