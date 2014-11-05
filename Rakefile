require 'rake/packagetask'
require 'rake/clean'
require 'json'
require 'pp'

CLEAN.include('build/**')

## Configuration
$mod = {
  name:     "Eagleshift-TantaresRebalanceOverhaul",
  url:      "https://raw.githubusercontent.com/eagleshift/TantaresRebalanceOverhaul/master/$avcfile$",
  path:     "GameData/Eagleshift/TantaresRebalanceOverhaul",
  version:  {
    major:  0,
    minor:  1,
    patch:  0,
    build:  nil,
  },
  change_log_url: "https://raw.githubusercontent.com/eagleshift/TantaresRebalanceOverhaul/v$version$/CHANGELOG.md",
  github:
  {
    username:           "eagleshift",
    repository:         "TantaresRebalanceOverhaul",
    allow_pre_release:  false,
  },
  ksp_version:
  {
    major:  0,
    minor:  25,
    patch:  0
  },
  ksp_version_min:
  {
    major:  0,
    minor:  24,
    patch:  0
  },
  ksp_version_max:
  {
    major:  nil,
    minor:  nil,
    patch:  nil
  },

  meta: {
    folder:   "Eagleshift/TantaresRebalanceOverhaul",
    files:  [
      FileList["src/**/**.cfg"],
    ].flatten,
    vendor: [
      FileList["vendor/**/**.*"],
    ].flatten,
    info: {
      files: [
        "README.md",
        "CHANGELOG.md",
      ],
      mod: [
        "Eagleshift-TantaresRebalanceOverhaul.version",
      ]
    }
  }
}

## *****************************************************************************

def version_string(data)
  [ data[:version][:major].nil? ? 0 : data[:version][:major],
    data[:version][:minor].nil? ? 0 : data[:version][:minor],
    data[:version][:patch].nil? ? 0 : data[:version][:patch],
    data[:version][:build]
  ].compact * "."
end

def generate_avc_file(avc_data)
  avc_converter = lambda do |h| 
    Hash === h ? 
      Hash[
        h.map do |k, v| 
          [k.to_s.upcase, avc_converter[v]] 
        end 
      ] : h 
  end

  JSON.pretty_generate(
    avc_converter[
      avc_data.deep_reject {
        |k,v| v.nil? or
        v == {} or
        [:meta].include? k
      }
    ]
  ).to_s.gsub(
    "$version$", "#{version_string(avc_data)}"
  ).gsub(
    "$avcfile$", "#{avc_data[:name]}.version"
  )
end

class Hash
  def deep_reject(&blk)
    self.dup.deep_reject!(&blk)
  end

  def deep_reject!(&blk)
    self.each do |k, v|
      v.deep_reject!(&blk)  if v.is_a?(Hash)
      self.delete(k)  if blk.call(k, v)
    end
  end
end


## *****************************************************************************

namespace :mod do
  desc "Generates the KSP AVC version file"
  task :generateavc do
    File.write("#{$mod[:name]}.version", generate_avc_file($mod))
  end

  desc "Builds a release package"
  task :release => [:clean, :generateavc] do |p|
    package = "release/#{$mod[:name]}_v#{version_string($mod)}.zip"
    zip_command = '7za a -tzip'

    filemap = {}
    [ $mod[:meta][:files],
      $mod[:meta][:info][:mod] ].flatten.each do |infile|
      filemap[infile] = "build/GameData/#{$mod[:meta][:folder]}/#{infile.sub 'src/', ''}"
    end
    $mod[:meta][:info][:files].each do |infile|
      filemap[infile] = "build/#{infile.sub 'src/', ''}"
    end
    $mod[:meta][:vendor].each do |infile|
      filemap[infile] = "build/GameData/#{infile.sub 'vendor/', ''}"
    end
    filemap.each do |infile, outfile|
      FileUtils.mkdir_p(File.dirname(outfile))
      cp infile, outfile
    end
    Dir.chdir('build') do
      sh "#{zip_command} ../#{package} *"
    end
  end
end
